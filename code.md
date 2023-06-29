~~~java
/**
 * 出入库记录Service业务层处理
 *
 * @author renzhezhe
 * @date 2022-05-19
 */
@Service
public class TInOutRecordServiceImpl extends ServiceImpl<TInOutRecordMapper, TInOutRecord> implements ITInOutRecordService, IDefualtService<TInOutRecord> {
    @Autowired
    private TInOutRecordMapper tInOutRecordMapper;

    @Autowired
    private TInOutRecordBodyMapper tInOutRecordBodyMapper;

    @Autowired
    private TStorehouseDetailsMapper tStorehouseDetailsMapper;
    @Autowired
    private TProductInformationMapper tProductInformationMapper;
    @Autowired
    private TMaterialPickingListNewMapper tMaterialPickingListNewMapper;
    @Autowired
    private TMaterialPickingNewMapper tMaterialPickingNewMapper;

    @Autowired
    private TPurchaseOrderHeaderMapper tPurchaseOrderHeaderMapper;

    @Autowired
    private TGoodsMapper tGoodsMapper;
    @Autowired
    private TMaterialInfoMapper tMaterialInfoMapper;
    @Autowired
    private SupplierMapper supplierMapper;
    @Autowired
    private TMaterialSupplierService tMaterialSupplierService;
    @Autowired
    private RedisCache redisCache;
    @Autowired
    private TInventoryDetailsService tInventoryDetailsService;
    @Autowired
    private ITMaterialPickingNewService materialPickingNewService;
    @Autowired
    private ITOrdersNewListService ordersNewListService;
    @Autowired
    private ITProductDrawingService productDrawingService;

    @Autowired
    private TProductionListNoService productionListNoService;
    @Autowired
    private TProductionOrderListMapper productionOrderListMapper;
    @Autowired
    private TTechnologicalProcessMapper technologicalProcessMapper;
    @Autowired
    private SysUserMapper userMapper;
    @Autowired
    private TWorkTaskService workTaskService;
    @Autowired
    private TAppProcessRecordMapper appProcessRecordMapper;
    @Autowired
    private TAppProcessRecordBodyMapper appProcessRecordBodyMapper;
    @Autowired
    private TAppProcessOutputMapper appProcessOutputMapper;
    @Autowired
    private TInOutRecordBodyInspectMapper inOutRecordBodyInspectMapper;
    @Autowired
    private TProductionOrderMapper productionOrderMapper;
    @Autowired
    private ITProductInfoService productInfoService;
    @Autowired
    private ITCapacityService capacityService;
    @Autowired
    private ITCapacityUserService capacityUserService;
    @Autowired
    private ISysUserService userService;
    @Autowired
    private TAppProcessRecordService appProcessRecordService;
    @Autowired
    private TPurchaseSingleMapper purchaseSingleMapper;

    /**
     * 查询出入库记录
     *
     * @param id 出入库记录ID
     * @return 出入库记录
     */
    @Override
    public TInOutRecord selectTInOutRecordById(Long id) {
        TInOutRecord tInOutRecord = tInOutRecordMapper.selectTInOutRecordById(id);
        List<TInOutRecordBody> tInOutRecordBodies = tInOutRecordBodyMapper.selectList(new LambdaQueryWrapper<TInOutRecordBody>().eq(TInOutRecordBody::getTInOutRecordId, id));
        if (CollUtil.isNotEmpty(tInOutRecordBodies)) {
            tInOutRecord.setInOutRecordBodyList(tInOutRecordBodies);
            for (TInOutRecordBody tInOutRecordBody : tInOutRecordBodies) {
                List<TInOutRecordBodyInspect> tInOutRecordBodyInspects = inOutRecordBodyInspectMapper.selectList(
                        Wrappers.lambdaQuery(TInOutRecordBodyInspect.class)
                                .eq(TInOutRecordBodyInspect::getInOutRecordBodyId, tInOutRecordBody.getId())
                );
                String identifier = tInOutRecordBody.getIdentifier();
                String[] split = StrUtil.split(identifier, ",");
                tInOutRecordBody.setIdentifierList(Arrays.asList(split));
                tInOutRecordBody.setBodyInspects(tInOutRecordBodyInspects);

            }
        }
        return tInOutRecord;
    }

    /**
     * 查询出入库记录列表
     *
     * @param tInOutRecord 出入库记录
     * @return 出入库记录
     */
    @Override
    public List<TInOutRecord> selectTInOutRecordList(TInOutRecord tInOutRecord) {
        return tInOutRecordMapper.selectTInOutRecordList(tInOutRecord);
    }

    /**
     * 新增出入库记录
     *
     * @param tInOutRecord 出入库记录
     * @return 结果
     */
    @Override
    @Transactional
    public int insertTInOutRecord(TInOutRecord tInOutRecord) {
        double sum = tInOutRecord.getInOutRecordBodyList().stream().mapToDouble(a -> Convert.toDouble(a.getQuantityInventory())).sum();
        if(sum == 0) {
            throw new RuntimeException("入库材料数量不能全部为0！");
        }
        String singleNumber = tInOutRecord.getSingleNumber();
        // 如果出库单号不为空，则校验唯一，否则自动生成
        if (StrUtil.isNotBlank(singleNumber)) {
            LambdaQueryWrapper<TInOutRecord> qw = new LambdaQueryWrapper<>();
            qw.eq(TInOutRecord::getSingleNumber, singleNumber)
                    .eq(TInOutRecord::getDelFlag, "0")
                    .eq(TInOutRecord::getOperateType, tInOutRecord.getOperateType());
            int count = count(qw);
            if (count > 0) {
                throw new RuntimeException("出库单号已存在，请勿重复操作！");
            }
        } else {
            tInOutRecord.setSingleNumber(createBatchNumberByDocumentType("inOutRecordNo", 1, 4).get(0));
        }
        getBasis(tInOutRecord, types.ADD);
        //判断是哪种类型  材料入库还是出库，产品入库还是出库
        switch (tInOutRecord.getOperateType()) {
            //采购入库或退料入库 其他入库
            case "0":
            case "1":
            case "8":
                purchaseInsert(tInOutRecord);
                break;
            case "2":
            case "4":
            case "7":
                //生产入库或退货入库 其他入库
                productInsert(tInOutRecord);
                break;
            // 领料出库
            case "5":
                purchaseOut(tInOutRecord);
                break;
            //成品出库
            case "6":
                // 成品出库 新建出库单时不扣减库存，运输单状态为运输中时才开始扣减库存，运输单作废，库存要加回去
//                 productOut(tInOutRecord);
                tInOutRecord.setOutboundOrderStatus("0");
                break;
            case "9":
                // 成品采购入库
                productPurchaseIn(tInOutRecord);
                break;
            default:
                break;
        }
        tInOutRecordMapper.insertTInOutRecord(tInOutRecord);
        //不同类型有各自不同的处理方法
        for (TInOutRecordBody tInOutRecordBody : tInOutRecord.getInOutRecordBodyList()) {
            if(new BigDecimal(0).compareTo(tInOutRecordBody.getQuantityInventory()) >= 0) {
                continue;
            }
            if(StrUtil.equals(tInOutRecord.getOperateType(), "6")) {
                tProductInformationMapper.updateOutRecordFLag(tInOutRecordBody.getIdentifier(), "1");
                // 销售订单 明细加上已出库数量
                ordersNewListService.subOrAddOutQuantity(tInOutRecordBody, true);
            }
            if(StrUtil.equals(tInOutRecord.getOperateType(), "0")) {
                // 修改采购单 已入库的数量
                TPurchaseSingle single = new TPurchaseSingle();
                single.setTicketNo(tInOutRecord.getAssociatedNo());
                single.setGoodsNo(tInOutRecordBody.getModel());
                single.setGoodsName(tInOutRecordBody.getName());
                single.setSpecifications(tInOutRecordBody.getSpecification());
                single.setUnitPrice(tInOutRecordBody.getUnit());
                single.setInStorageQuantity(tInOutRecordBody.getQuantityInventory());
                purchaseSingleMapper.updateInStorageQuantity(single);
            }
            tInOutRecordBody.setTInOutRecordId(tInOutRecord.getId());
            tInOutRecordBody.setCreateTime(tInOutRecord.getCreateTime());
            tInOutRecordBody.setCreateBy(tInOutRecord.getCreateBy());
            tInOutRecordBody.setUpdateBy(tInOutRecord.getUpdateBy());
            tInOutRecordBody.setUpdateTime(tInOutRecord.getUpdateTime());
            // 领料出库， 出库单和领料单一对多。一个出库单明细对应一条领料单
            if(!StrUtil.equals(tInOutRecord.getOperateType(), "5") && !StrUtil.equals(tInOutRecord.getOperateType(), "6")) {
                tInOutRecordBody.setAssociatedNo(tInOutRecord.getAssociatedNo());
            } else {
                // 领料出库
                List<SupplierStoreSubDTO> setNumList = tInOutRecordBody.getSetNumList();
                if(CollUtil.isNotEmpty(setNumList)) {
                    tInOutRecordBody.setOutSupplier(JSON.toJSONString(setNumList));
                }
            }
            if(tInOutRecord.getOperateType().equals("9")) {
                List<String> identifierList = tInOutRecordBody.getIdentifierList();
                String collect = String.join(",", identifierList);
                tInOutRecordBody.setIdentifier(collect);
            }
            tInOutRecordBody.setOperateType(tInOutRecord.getOperateType());
            tInOutRecordBodyMapper.insertTInOutRecordBody(tInOutRecordBody);
            if(tInOutRecord.getOperateType().equals("9")) {
                List<String> identifierList = tInOutRecordBody.getIdentifierList();
                for (String identifier : identifierList) {
                    TInOutRecordBodyInspect bodyInspect = new TInOutRecordBodyInspect();
                    bodyInspect.setInOutRecordBodyId(tInOutRecordBody.getId());
                    bodyInspect.setCreateBy(SecurityUtils.getUsername());
                    bodyInspect.setIdentifier(identifier);
                    bodyInspect.setStatus("0");
                    inOutRecordBodyInspectMapper.insert(bodyInspect);
                }
            }
        }
        //如果是采购订单修改状态
        if(tInOutRecord.getOperateType().equals("0")|| tInOutRecord.getOperateType().equals("9")){
            String associatedNo = tInOutRecord.getAssociatedNo();
            List<TPurchaseSingleDetailVo> tPurchaseSingleDetailByTicketNo = tInOutRecordMapper.getTPurchaseSingleDetailByTicketNo(associatedNo);
            if(tInOutRecord.getOperateType().equals("9")) {
                tPurchaseSingleDetailByTicketNo = tInOutRecordMapper.getTProductSingleDetailByTicketNo(associatedNo);
            }
            TPurchaseOrderHeader tPurchaseOrderHeader = new TPurchaseOrderHeader();
            tPurchaseOrderHeader.setTicketNo(associatedNo);
            BigDecimal need = new BigDecimal(0);
            BigDecimal haved = new BigDecimal(0);
            for(TPurchaseSingleDetailVo tPurchaseSingleDetailVo:tPurchaseSingleDetailByTicketNo){
                need = need.add(tPurchaseSingleDetailVo.getQuantityInventory());
                haved = haved.add(tPurchaseSingleDetailVo.getLaveQuantityInventoryed());
            }
            if(haved.intValue()==0){
                tPurchaseOrderHeader.setProductStatus("0");
            }else if(haved.intValue() >= need.intValue()){
                tPurchaseOrderHeader.setProductStatus("1");
            }else{
                tPurchaseOrderHeader.setProductStatus("2");
            }
            tPurchaseOrderHeaderMapper.updateTPurchaseOrderHeader(tPurchaseOrderHeader);
        }
        return 1;
    }

    /**
     * 根据运输清单扣减库存
     * @param goods
     */
    @Override
    @Transactional
    public synchronized void productOutByIdentifier(List<TGoods> goods) {
        for (TGoods good : goods) {
            // 成品唯一识别码
            String identifier = good.getBatchNumber();
            // 扣减库存明细库存
            TInventoryDetails inventoryDetails = tInventoryDetailsService.getOne(
                    Wrappers.lambdaQuery(TInventoryDetails.class).eq(TInventoryDetails::getIdentifier, identifier)
            );
            if(inventoryDetails == null) {
                throw new RuntimeException("库存中不存在'" + good.getGoodsName() + "'产品，请选择其他出库单！");
            }
            BigDecimal inventoryQuantity = inventoryDetails.getInventoryQuantity();
            if (inventoryQuantity.compareTo(NumberUtil.toBigDecimal(1)) < 0) {
                // 库存不足，出库失败
                throw new RuntimeException("产品：'" + good.getGoodsName() + "' 库存不足，请选择其他出库单运输！");
            }
            inventoryDetails.setInventoryQuantity(NumberUtil.sub(inventoryQuantity, NumberUtil.toBigDecimal(1)));
            inventoryDetails.setOutStoreDate(new Date());
            tInventoryDetailsService.updateById(inventoryDetails);
            // 扣减库存主表库存数量  根据 产品编码，名称，规格查询
            TStorehouseDetails tStorehouseDetails = tStorehouseDetailsMapper.selectOne(
                    Wrappers.lambdaQuery(TStorehouseDetails.class)
                            .eq(TStorehouseDetails::getName, inventoryDetails.getName())
                            .eq(TStorehouseDetails::getModel, inventoryDetails.getModel())
                            .eq(TStorehouseDetails::getSpecification, inventoryDetails.getSpecification())
            );
            if(tStorehouseDetails == null) {
                throw new RuntimeException("库存中不存在'" + good.getGoodsName() + "'产品，请选择其他出库单！");
            }
            BigDecimal inventoryQuantity1 = tStorehouseDetails.getInventoryQuantity();
            if (inventoryQuantity1.compareTo(NumberUtil.toBigDecimal(1)) < 0) {
                // 库存不足，出库失败
                throw new RuntimeException("产品：'" + good.getGoodsName() + "' 库存不足，请选择其他出库单！");
            }
            tStorehouseDetails.setInventoryQuantity(NumberUtil.sub(inventoryQuantity1, NumberUtil.toBigDecimal(1)));
            tStorehouseDetailsMapper.updateById(tStorehouseDetails);
        }
    }

    @Override
    @Transactional
    public synchronized void productInByIdentifier(List<TGoods> goods) {
        for (TGoods good : goods) {
            // 成品唯一识别码
            String identifier = good.getBatchNumber();
            TInventoryDetails inventoryDetails = tInventoryDetailsService.getOne(
                    Wrappers.lambdaQuery(TInventoryDetails.class).eq(TInventoryDetails::getIdentifier, identifier)
            );
            if(inventoryDetails != null) {
                BigDecimal inventoryQuantity = inventoryDetails.getInventoryQuantity();
                inventoryDetails.setInventoryQuantity(NumberUtil.add(inventoryQuantity, NumberUtil.toBigDecimal(1)));
                tInventoryDetailsService.updateById(inventoryDetails);
            }
            TStorehouseDetails tStorehouseDetails = tStorehouseDetailsMapper.selectOne(
                    Wrappers.lambdaQuery(TStorehouseDetails.class)
                            .eq(TStorehouseDetails::getName, inventoryDetails.getName())
                            .eq(TStorehouseDetails::getModel, inventoryDetails.getModel())
                            .eq(TStorehouseDetails::getSpecification, inventoryDetails.getSpecification())
            );
            if(tStorehouseDetails != null) {
                BigDecimal inventoryQuantity = tStorehouseDetails.getInventoryQuantity();
                tStorehouseDetails.setInventoryQuantity(NumberUtil.add(inventoryQuantity, NumberUtil.toBigDecimal(1)));
                tStorehouseDetailsMapper.updateById(tStorehouseDetails);
            }
        }
    }

    /**
     * 产品出库
     */
    private synchronized void productOut(TInOutRecord tInOutRecord) {
        //通过唯一标识出库
        List<TInOutRecordBody> inOutRecordBodyList = tInOutRecord.getInOutRecordBodyList();
        if (CollUtil.isNotEmpty(inOutRecordBodyList)) {
            for (TInOutRecordBody tInOutRecordBody : inOutRecordBodyList) {
                String identifier = tInOutRecordBody.getIdentifier();
                // 减去库存
                BigDecimal quantityInventory = tInOutRecordBody.getQuantityInventory();
                TStorehouseDetails tStorehouseDetails = tStorehouseDetailsMapper.selectOne(
                        new LambdaQueryWrapper<TStorehouseDetails>()
                                .eq(TStorehouseDetails::getDelFlag, "0")
                                .eq(TStorehouseDetails::getIdentifier, identifier)
                );
                if (tStorehouseDetails != null) {
                    Integer count = tInOutRecordBodyMapper.selectCount(
                            new LambdaQueryWrapper<TInOutRecordBody>()
                                    .eq(TInOutRecordBody::getIdentifier, identifier)
                                    .eq(TInOutRecordBody::getDelFlag, "0")
                                    .eq(TInOutRecordBody::getOperateType, tInOutRecord.getOperateType())
                    );
                    if (count != 0) {
                        throw new RuntimeException("您选择的成品：'" + tStorehouseDetails.getName() + "',唯一识别号：" + tStorehouseDetails.getIdentifier() + ", 已存在出库单，请勿重复操作！");
                    }
                    BigDecimal inventoryQuantity = tStorehouseDetails.getInventoryQuantity();
                    if (inventoryQuantity.compareTo(quantityInventory) < 0) {
                        // 库存不足，出库失败
                        throw new RuntimeException("产品：'" + tInOutRecordBody.getName() + "' 库存不足，出库失败！");
                    }
                    tStorehouseDetails.setInventoryQuantity(NumberUtil.sub(inventoryQuantity, quantityInventory));
                    tStorehouseDetailsMapper.updateById(tStorehouseDetails);
                } else {
                    throw new RuntimeException("库存中不存在'" + tInOutRecordBody.getName() + "'产品，出库失败！");
                }
            }
        }
    }

    /**
     * 材料出库
     */
    private synchronized void purchaseOut(TInOutRecord tInOutRecord) {
        List<TInOutRecordBody> inOutRecordBodyList = tInOutRecord.getInOutRecordBodyList();
        Map<String, List<TInOutRecordBody>> map = inOutRecordBodyList.stream().filter(a -> new BigDecimal(0).compareTo(a.getActQuantity()) < 0).collect(Collectors.groupingBy(TInOutRecordBody::getAssociatedNo));
        for (String associateNo : map.keySet()) {
            // 回填领料单信息
            TMaterialPickingNew tpn = new TMaterialPickingNew();
            tpn.setManager(tInOutRecord.getReviewer());
            tpn.setReviewer(tInOutRecord.getApplicant());
            tpn.setDateTime(DateUtil.format(new Date(), "yyyy-MM-dd"));
            tpn.setNo(associateNo);
            //如果点击出库单的备货按钮 则会传0，正常材料出库字段为空 cj-20230328
            if(null!=tInOutRecord.getOutboundOrderStatus()&&"0".equals(tInOutRecord.getOutboundOrderStatus())){
                tpn.setProductStatus("2");//将领料单状态修改为已备货
            }else{
                tpn.setProductStatus("");
            }
            materialPickingNewService.updateByNo(tpn);
            // 出库
            List<TInOutRecordBody> tInOutRecordBodies = map.get(associateNo);
            for (TInOutRecordBody tInOutRecordBody : tInOutRecordBodies) {
                String model = tInOutRecordBody.getModel();
                String name = tInOutRecordBody.getName();
                String specification = tInOutRecordBody.getSpecification();
                String unit = tInOutRecordBody.getUnit();
                // 实际出库总数量
                BigDecimal actQuantity = tInOutRecordBody.getActQuantity();
                // 出库， 库存主表，判断库存是否充足。如果充足，主表直接减数量，明细 根据 入库时间 先进先出规则扣减库存数量
                // 1. 主表库存是否充足
                LambdaQueryWrapper<TStorehouseDetails> qw = Wrappers
                        .lambdaQuery(TStorehouseDetails.class)
                        .eq(TStorehouseDetails::getModel, model)
                        .eq(TStorehouseDetails::getName, name)
                        .eq(TStorehouseDetails::getSpecification, specification)
                        .eq(TStorehouseDetails::getUnit, unit)
                        .ge(TStorehouseDetails::getInventoryQuantity, actQuantity);
                TStorehouseDetails tStorehouseDetails = tStorehouseDetailsMapper.selectOne(qw);
                if (tStorehouseDetails == null) {
                    String msg = StrUtil.format("编码:{}, 名称:{}, 规格:{}, 单位:{}, 的材料库存不足，出库失败！", model, name, specification, unit);
                    throw new RuntimeException(msg);
                }
                // 2. 扣减主表库存数量
                BigDecimal num = NumberUtil.sub(tStorehouseDetails.getInventoryQuantity(), actQuantity);
                tStorehouseDetails.setInventoryQuantity(num);
                tStorehouseDetailsMapper.updateById(tStorehouseDetails);
                // 3. 库存明细表根据 入库时间 先进先出规则扣减库存数量
                List<TInventoryDetails> inventoryDetails = tInventoryDetailsService.getInventoryDetails(model, name, specification, tInOutRecord.getInventoryType());
                if (CollUtil.isEmpty(inventoryDetails)) {
                    String msg = StrUtil.format("编码:{}, 名称:{}, 规格:{}, 单位:{}, 的材料库存不足，出库失败！", model, name, specification, unit);
                    throw new RuntimeException(msg);
                }
                for (TInventoryDetails inventoryDetail : inventoryDetails) {
                    BigDecimal inventoryQuantity = inventoryDetail.getInventoryQuantity();
                    BigDecimal outQuantity = inventoryDetail.getOutQuantity();
                    if (actQuantity.compareTo(inventoryQuantity) <= 0) {
                        inventoryDetail.setInventoryQuantity(NumberUtil.sub(inventoryQuantity, actQuantity));
                        // 已出库数量
                        inventoryDetail.setOutQuantity(NumberUtil.add(outQuantity, actQuantity));
                        tInventoryDetailsService.updateById(inventoryDetail);
                        break;
                    } else {
                        actQuantity = NumberUtil.sub(actQuantity, inventoryQuantity);
                        // 已出库数量
                        inventoryDetail.setOutQuantity(NumberUtil.add(outQuantity, inventoryQuantity));
                        inventoryDetail.setInventoryQuantity(NumberUtil.toBigDecimal(0));
                        tInventoryDetailsService.updateById(inventoryDetail);
                    }
                }

                // 将 actQuantity 实际领料数量 更新到对应的领料单下的对应的材料下，
                // 该材料初始的 returnableQuantity（可退料数量） 等于 actQuantity
                // 要唯一确定该材料，需要 no、productName、model、specification、unit 字段
                // tInOutRecordBody 中的 associateNo 对应 tMaterialPickingListNew 的 no
                // tInOutRecordBody 中的 model 对应 tMaterialPickingListNew 的 componentItemNumber
                LambdaQueryWrapper<TMaterialPickingListNew> queryWrapper = Wrappers
                        .lambdaQuery(TMaterialPickingListNew.class)
                        .eq(TMaterialPickingListNew::getNo, tInOutRecordBody.getAssociatedNo())
                        .eq(TMaterialPickingListNew::getProductName, tInOutRecordBody.getName())
                        .eq(TMaterialPickingListNew::getComponentItemNumber, tInOutRecordBody.getModel())
                        .eq(TMaterialPickingListNew::getSpecification, tInOutRecordBody.getSpecification())
                        .eq(TMaterialPickingListNew::getUnit, tInOutRecordBody.getUnit());
                TMaterialPickingListNew tMaterialPickingListNew = tMaterialPickingListNewMapper.selectOne(queryWrapper);
                // 可退还数量 初始化为 实际领料数量
                tMaterialPickingListNew.setActQuantity(tInOutRecordBody.getActQuantity());
                tMaterialPickingListNew.setReturnableQuantity(tInOutRecordBody.getActQuantity());
                tMaterialPickingListNewMapper.updateTMaterialPickingListNew(tMaterialPickingListNew);
            }
            TMaterialPickingNew tMaterialPickingNew = tMaterialPickingNewMapper.selectOne(new LambdaQueryWrapper<TMaterialPickingNew>().eq(TMaterialPickingNew::getNo, associateNo));
            if (tMaterialPickingNew != null) {
                tMaterialPickingNew.setProductStatus("1");
                tMaterialPickingNewMapper.updateById(tMaterialPickingNew);
            }
        }
    }

    /**
     * 成品采购入库
     * @param tInOutRecord
     */
    private synchronized void productPurchaseIn(TInOutRecord tInOutRecord){
        for (TInOutRecordBody tInOutRecordBody : tInOutRecord.getInOutRecordBodyList()) {
            Long productOrderListId = tInOutRecordBody.getProductOrderListId();
            TProductionOrderList orderList = productionOrderListMapper.selectById(productOrderListId);
            // 修改生产订单状态进行中
            String productionOrderCode = orderList.getProductionOrderCode();
            productionOrderMapper.updateProductionOrderStatus(productionOrderCode);

            String projectName = orderList.getProjectName();
            String customerName = orderList.getCustomerName();
            String contractNo = orderList.getContractNo();
            // 成品采购入库不直接入库，自动生成任务单-加工记录，后去质检再入库
            TWorkTask workTask = new TWorkTask();
            // 不让列表中查询到
            workTask.setDelFlag("2");
            workTask.setDispatchPerson("系统管理员");
            workTask.setDispatchTime(new Date());
            TWorkTaskProduction production = new TWorkTaskProduction();
            production.setBakOne(Convert.toStr(productOrderListId));
            // 图号
            String model = tInOutRecordBody.getModel();
            List<TProductDrawing> tProductDrawings = productDrawingService.selectTProductDrawingListByProductCode(model);
            TProductDrawing tProductDrawing = tProductDrawings.get(0);
            production.setDrawingNo(tProductDrawing.getDrawingCode());
            production.setPlanStartTime(new Date());
            production.setPlanEndTime(new Date());
            production.setProductCode(model);
            production.setProductName(tInOutRecordBody.getName());
            production.setSpecification(tInOutRecordBody.getSpecification());
            production.setQuantity(Convert.toInt(tInOutRecordBody.getQuantityInventory()));
            production.setUnit(tInOutRecordBody.getUnit());
            production.setProductionCode(orderList.getProductionOrderCode());
            workTask.setTWorkTaskProduction(production);
            TWorkTaskList taskList = new TWorkTaskList();
            taskList.setBakOne("1");
            taskList.setQuantity(Convert.toInt(tInOutRecordBody.getQuantityInventory()));
            taskList.setDispatchedQuantity(Convert.toInt(tInOutRecordBody.getQuantityInventory()));
            taskList.setNeedQuantity(Convert.toInt(tInOutRecordBody.getQuantityInventory()));
            taskList.setPlanStartTime(new Date());
            taskList.setPlanEndTime(new Date());
            // 根据图号查询最后一道工艺
            List<TTechnologicalProcess> processes = technologicalProcessMapper.selectTTechnologicalProcessListByDrawingCode(tProductDrawing.getDrawingCode());
            List<TTechnologicalProcess> processList = processes.stream().filter(a -> StrUtil.equals("0", a.getIsStopProcess())).collect(Collectors.toList());
            if(CollUtil.isEmpty(processList)) {
                throw new RuntimeException("未找到该产品的最后一道工序！");
            }
            TTechnologicalProcess process = processList.get(0);
            String roles = process.getRoles();
            String operatorPerson = "";
            if(StrUtil.isNotBlank(roles)) {
                String[] rolesArr = roles.split(",");
                List<SysUser> userByRoleIds = userMapper.getUserByRoleIds(rolesArr);
                if(CollUtil.isNotEmpty(userByRoleIds)) {
                    operatorPerson = userByRoleIds.get(0).getJobNumber();
                    taskList.setOperatorPerson(operatorPerson);
                    taskList.setOperatorPersonList(userByRoleIds);
                }
            }
            taskList.setProcessName(process.getProcessName());
            taskList.setProcessCode(process.getProcessCode());
            List<String> identifierList = tInOutRecordBody.getIdentifierList();
            List<TWorkTaskListNo> workTaskListNos = identifierList.stream().map(a -> {
                TWorkTaskListNo no = new TWorkTaskListNo();
                no.setContractNo(contractNo);
                no.setProjectName(projectName);
                no.setCustomerName(customerName);
                no.setIdentifier(a);
                no.setProductionOrderListId(productOrderListId);
                return no;
            }).collect(Collectors.toList());
            taskList.setWorkTaskListNos(workTaskListNos);
            List<TWorkTaskList> list = new ArrayList<>();
            list.add(taskList);
            workTask.setTWorkTaskLists(list);
            workTask.setCreatePickingFlag("0");
            workTaskService.addTask(workTask);
            // 生成加工记录主表
            for (String s : identifierList) {
                TAppProcessRecord record = new TAppProcessRecord();
                record.setTaskNo(workTask.getTaskNo());
                List<TWorkTaskList> tWorkTaskLists = workTask.getTWorkTaskLists();
                TWorkTaskList taskList1 = tWorkTaskLists.get(0);
                record.setWorkTaskListId(taskList1.getId());
                record.setProcessCode(taskList1.getProcessCode());
                record.setProcessName(taskList1.getProcessName());
                record.setIdentifier(s);
                record.setOperatorPerson(operatorPerson);
                record.setOperatorTime(new Date());
                // 如果该工序不需要质检，则跳过质检，直接编程质检通过
                if(StrUtil.equals("1", process.getIsInspect())) {
                    record.setStatus("2");
                } else {
                    record.setStatus("0");
                }
                record.setCreateBy(SecurityUtils.getUsername());
                appProcessRecordMapper.insert(record);
                // 加工记录明细
                TAppProcessRecordBody body = new TAppProcessRecordBody();
                body.setProcessRecordId(record.getId());
                body.setOperatorPerson(operatorPerson);
                body.setOperatorTime(new Date());
                body.setCreateBy(SecurityUtils.getUsername());
                appProcessRecordBodyMapper.insert(body);
                TAppProcessOutput output = new TAppProcessOutput();
                output.setProcessRecordId(record.getId());
                output.setOutputProductNum(tInOutRecordBody.getModel());
                output.setOutputProductName(tInOutRecordBody.getName());
                output.setIdentifier(s);
                output.setSpecification(tInOutRecordBody.getSpecification());
                output.setCreateBy(SecurityUtils.getUsername());
                appProcessOutputMapper.insert(output);
            }
        }
    }

    /**
     * 成品采购入库
     * @param tInOutRecord
     */
    private synchronized void productPurchaseIn1(TInOutRecord tInOutRecord){
        List<TInOutRecordBody> inOutRecordBodyList = tInOutRecord.getInOutRecordBodyList();
        List<TStorehouseDetails> tStorehouseDetails = new ArrayList<>();
        // 库存明细
        List<TProductionListNo> list = new ArrayList<>();
        List<TInventoryDetails> inventoryDetailsList = new ArrayList<>();
        if (CollUtil.isNotEmpty(inOutRecordBodyList)) {
            Map<String, InOutRecordVo> totalMap = new HashMap<>();
            for (TInOutRecordBody tInOutRecordBody : inOutRecordBodyList) {
                if (new BigDecimal(0).compareTo(tInOutRecordBody.getQuantityInventory()) >= 0) {
                    continue;
                }
                String name = tInOutRecordBody.getName();
                String model = tInOutRecordBody.getModel();
                String specification = tInOutRecordBody.getSpecification();
                //key
                String key = name + ":" + specification + ":" + model;
                InOutRecordVo inOutRecordVo = totalMap.get(key);
                BigDecimal quantityInventory = tInOutRecordBody.getQuantityInventory();
                int quantity = Convert.toInt(quantityInventory);
                List<String> identifierList = tInOutRecordBody.getIdentifierList();
                if(quantity != identifierList.size()) {
                    throw new RuntimeException("入库数量和成品编号数量不等，请检查数据！");
                }
                if (inOutRecordVo == null) {
                    inOutRecordVo = new InOutRecordVo();
                    inOutRecordVo.setName(tInOutRecordBody.getName());
                    inOutRecordVo.setModel(tInOutRecordBody.getModel());
                    inOutRecordVo.setSpecification(specification);
                    inOutRecordVo.setType(tInOutRecordBody.getType());
                    inOutRecordVo.setInventoryQuantity(quantityInventory);
                    inOutRecordVo.setUnit(tInOutRecordBody.getUnit());
                    totalMap.put(key, inOutRecordVo);
                } else {
                    inOutRecordVo.setInventoryQuantity(NumberUtil.add(inOutRecordVo.getInventoryQuantity(), new BigDecimal(1)));
                }
                for (String identifier : identifierList) {
                    // 库存明细
                    TInventoryDetails inventoryDetails = new TInventoryDetails();
                    inventoryDetails.setName(name);
                    inventoryDetails.setModel(model);
                    inventoryDetails.setSpecification(specification);
                    inventoryDetails.setUnit(tInOutRecordBody.getUnit());
                    inventoryDetails.setInventoryQuantity(new BigDecimal(1));
                    inventoryDetails.setInventoryType(Convert.toInt(tInOutRecord.getInventoryType()));
                    inventoryDetails.setInventoryDate(tInOutRecord.getWarehousingTime());
                    inventoryDetails.setRemark(tInOutRecordBody.getRemark());
                    inventoryDetails.setIdentifier(identifier);
                    inventoryDetailsList.add(inventoryDetails);
                    TProductionListNo no = new TProductionListNo();
                    no.setIdentifier(identifier);
                    no.setProductionListId(tInOutRecordBody.getProductOrderListId());
                    no.setCreateBy(SecurityUtils.getLoginUser().getUser().getNickName());
                    list.add(no);
                }
            }
            for (String s : totalMap.keySet()) {
                TStorehouseDetails tStorehouseDetail = new TStorehouseDetails();
                InOutRecordVo inOutRecordVo = totalMap.get(s);
                /** 库存类型;0-产品 1-材料 */
                tStorehouseDetail.setInventoryType(tInOutRecord.getInventoryType());
                /** 分类 分类 0-采购入库 1-退料入库 2-生产入库 3-退货入库 */
                tStorehouseDetail.setType(tInOutRecord.getOperateType());
                tStorehouseDetail.setModel(inOutRecordVo.getModel());
                tStorehouseDetail.setName(inOutRecordVo.getName());
                tStorehouseDetail.setSpecification(inOutRecordVo.getSpecification());
                tStorehouseDetail.setUnit(inOutRecordVo.getUnit());
                tStorehouseDetail.setType(inOutRecordVo.getType());
                /** 库存数量 */
                tStorehouseDetail.setInventoryQuantity(inOutRecordVo.getInventoryQuantity());
                tStorehouseDetails.add(tStorehouseDetail);
            }
            if (CollUtil.isNotEmpty(tStorehouseDetails)) {
                for (TStorehouseDetails tStorehouseDetail : tStorehouseDetails) {
                    //查询条件为 仓库id+名称+型号+
                    TStorehouseDetails tStorehouseDetails1 = tStorehouseDetailsMapper.selectOne(
                            new LambdaQueryWrapper<TStorehouseDetails>()
                                    .eq(TStorehouseDetails::getName, tStorehouseDetail.getName())
                                    .eq(TStorehouseDetails::getModel, tStorehouseDetail.getModel())
                                    .eq(TStorehouseDetails::getSpecification, tStorehouseDetail.getSpecification())
                    );
                    if (tStorehouseDetails1 == null) {
                        tStorehouseDetailsMapper.insert(tStorehouseDetail);
                    } else {
                        tStorehouseDetails1.setInventoryQuantity(NumberUtil.add(tStorehouseDetail.getInventoryQuantity(), tStorehouseDetails1.getInventoryQuantity()));
                        tStorehouseDetailsMapper.updateById(tStorehouseDetails1);
                    }
                }
                // 插入库存明细
                if (CollUtil.isNotEmpty(inventoryDetailsList)) {
                    tInventoryDetailsService.saveBatch(inventoryDetailsList);
                }
            }
            // 插入 产品编号和生成订单明细关联
            if(CollUtil.isNotEmpty(list)) {
                productionListNoService.saveBatch(list);
            }
        } else {
            throw new RuntimeException("入库清单不能为空！");
        }
    }

    /**
     * 生产入库或退货入库
     */
    private synchronized void productInsert(TInOutRecord tInOutRecord) {
        List<TInOutRecordBody> inOutRecordBodyList = tInOutRecord.getInOutRecordBodyList();
        List<TStorehouseDetails> tStorehouseDetails = new ArrayList<>();
        // 库存明细
        List<TInventoryDetails> inventoryDetailsList = new ArrayList<>();
        HashSet<String> hashSet = new HashSet<>();
        if (CollUtil.isNotEmpty(inOutRecordBodyList)) {
            Map<String, InOutRecordVo> totalMap = new HashMap<>();
            for (TInOutRecordBody tInOutRecordBody : inOutRecordBodyList) {
                if (new BigDecimal(0).compareTo(tInOutRecordBody.getQuantityInventory()) >= 0) {
                    continue;
                }
                String name = tInOutRecordBody.getName();
                String model = tInOutRecordBody.getModel();
                String specification = tInOutRecordBody.getSpecification();
                //key
                String key = name + ":" + specification + ":" + model;
                InOutRecordVo inOutRecordVo = totalMap.get(key);
                if (inOutRecordVo == null) {
                    inOutRecordVo = new InOutRecordVo();
                    inOutRecordVo.setName(tInOutRecordBody.getName());
                    inOutRecordVo.setModel(tInOutRecordBody.getModel());
                    inOutRecordVo.setSpecification(specification);
                    inOutRecordVo.setType(tInOutRecordBody.getType());
                    inOutRecordVo.setInventoryQuantity(new BigDecimal(1));
                    inOutRecordVo.setUnit(tInOutRecordBody.getUnit());
                    totalMap.put(key, inOutRecordVo);
                } else {
                    inOutRecordVo.setInventoryQuantity(NumberUtil.add(inOutRecordVo.getInventoryQuantity(), new BigDecimal(1)));
                }
                // 库存明细
                TInventoryDetails inventoryDetails = new TInventoryDetails();
                inventoryDetails.setName(name);
                inventoryDetails.setModel(model);
                inventoryDetails.setSpecification(specification);
                inventoryDetails.setUnit(tInOutRecordBody.getUnit());
                inventoryDetails.setInventoryQuantity(tInOutRecordBody.getQuantityInventory());
                inventoryDetails.setInventoryType(Convert.toInt(tInOutRecord.getInventoryType()));
                inventoryDetails.setInventoryDate(tInOutRecord.getWarehousingTime());
                inventoryDetails.setRemark(tInOutRecordBody.getRemark());
                inventoryDetails.setIdentifier(tInOutRecordBody.getIdentifier());
                inventoryDetailsList.add(inventoryDetails);
                hashSet.add(tInOutRecordBody.getIdentifier());
            }
            for (String s : totalMap.keySet()) {
                TStorehouseDetails tStorehouseDetail = new TStorehouseDetails();
                InOutRecordVo inOutRecordVo = totalMap.get(s);
                /** 库存类型;0-产品 1-材料 */
                tStorehouseDetail.setInventoryType(tInOutRecord.getInventoryType());
                /** 分类 分类 0-采购入库 1-退料入库 2-生产入库 3-退货入库 */
                tStorehouseDetail.setType(tInOutRecord.getOperateType());
                tStorehouseDetail.setModel(inOutRecordVo.getModel());
                tStorehouseDetail.setName(inOutRecordVo.getName());
                tStorehouseDetail.setSpecification(inOutRecordVo.getSpecification());
                tStorehouseDetail.setUnit(inOutRecordVo.getUnit());
                tStorehouseDetail.setType(inOutRecordVo.getType());
                /** 库存数量 */
                tStorehouseDetail.setInventoryQuantity(inOutRecordVo.getInventoryQuantity());
                tStorehouseDetails.add(tStorehouseDetail);
            }
            if (CollUtil.isNotEmpty(tStorehouseDetails)) {
                for (TStorehouseDetails tStorehouseDetail : tStorehouseDetails) {
                    //查询条件为 仓库id+名称+型号+
                    TStorehouseDetails tStorehouseDetails1 = tStorehouseDetailsMapper.selectOne(
                            new LambdaQueryWrapper<TStorehouseDetails>()
                                    .eq(TStorehouseDetails::getName, tStorehouseDetail.getName())
                                    .eq(TStorehouseDetails::getModel, tStorehouseDetail.getModel())
                                    .eq(TStorehouseDetails::getSpecification, tStorehouseDetail.getSpecification())
                    );
                    if (tStorehouseDetails1 == null) {
                        tStorehouseDetailsMapper.insert(tStorehouseDetail);
                    } else {
                        tStorehouseDetails1.setInventoryQuantity(NumberUtil.add(tStorehouseDetail.getInventoryQuantity(), tStorehouseDetails1.getInventoryQuantity()));
                        tStorehouseDetailsMapper.updateById(tStorehouseDetails1);
                    }
                }
                // 插入库存明细
                if (CollUtil.isNotEmpty(inventoryDetailsList)) {
                    tInventoryDetailsService.saveBatch(inventoryDetailsList);
                }
            }
            //将对于的成品标记成已入库
            tProductInformationMapper.updateStatus(hashSet);
        } else {
            throw new RuntimeException("入库清单不能为空！");
        }

        // 录入产能
        // 如果是生产入库，将该入库记录列入产能分析表t_Capacity
        if ("2".equals(tInOutRecord.getOperateType())) {
            // 从 tInOutRecord 中获取入库时间，并将时间切分为年/月/日
            Date warehousingTime = tInOutRecord.getWarehousingTime();
            SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
            String transformDate = format.format(warehousingTime);
            String year = transformDate.substring(0, 4);
            String month = transformDate.substring(5, 7);
            String day = transformDate.substring(8, 10);

            // 从 tInOutRecord 中获取 applicant (applicant 对应的是 SysUser 的 nickName)
            // 并通过 nickName 获取 jobNumber
            String userName = tInOutRecord.getApplicant();
            String jobNumber = userService.selectUserByNickName(userName).getJobNumber();

            inOutRecordBodyList.forEach(body -> {
                // 通过 TInOutRecordBody 的产品名称，产品编码，规格得到该产品的产品类型，并获取产品的单位和入库数量
                TProductInfo product = new TProductInfo();
                product.setProductName(body.getName());     // 产品名称
                product.setProductCode(body.getModel());    // 产品编码
                product.setProductSpecs(body.getSpecification());   // 产品规格
                product.setUnit(body.getUnit());    // 产品单位
                // 查询该产品的产品类型
                String productCategory = productInfoService.selectTProductInfoList(product).get(0).getProductCategory();
                // 插入产能记录
                TCapacity capacity = new TCapacity();
                capacity.setYear(year);
                capacity.setMonth(month);
                capacity.setDay(day);
                capacity.setProductCategory(productCategory);
                capacity.setUnit(body.getUnit());
                capacity.setQuantity(body.getQuantityInventory());
                capacityService.insertTCapacity(capacity);

                // 插入人均产能记录 -- 入库
                TCapacityUser capacityUser = new TCapacityUser();
                capacityUser.setYear(year);
                capacityUser.setMonth(month);
                capacityUser.setDay(day);
                // 生产入库的工艺名称统一规定为 入库
                capacityUser.setProcessName("入库");
                capacityUser.setUserName(userName);
                capacityUser.setJobNumber(jobNumber);
                capacityUser.setQuantity(body.getQuantityInventory());
                capacityUserService.insertTCapacityUser(capacityUser);
            });
        }
    }

    /**
     * 采购入库或退料入库
     */
    private synchronized void purchaseInsert(TInOutRecord tInOutRecord) {
        //采购入库，将数量对应添加到明细仓库中去
        List<TInOutRecordBody> inOutRecordBodyList = tInOutRecord.getInOutRecordBodyList();

        List<TStorehouseDetails> tStorehouseDetails = new ArrayList<>();
        // 库存明细
        List<TInventoryDetails> inventoryDetailsList = new ArrayList<>();
        if (CollUtil.isNotEmpty(inOutRecordBodyList)) {
            Map<String, InOutRecordVo> totalMap = new HashMap<>();
            for (TInOutRecordBody tInOutRecordBody : inOutRecordBodyList) {
                if(new BigDecimal(0).compareTo(tInOutRecordBody.getQuantityInventory()) >= 0) {
                    continue;
                }
                //名称
                String name = tInOutRecordBody.getName();
                //规格
                String specification = tInOutRecordBody.getSpecification();
                //型号
                String model = tInOutRecordBody.getModel();
                // 供应商
                // String supplierCode = tInOutRecordBody.getSupplierCode();
                //key
                String key = name + ":" + specification + ":" + model;
                InOutRecordVo inOutRecordVo = totalMap.get(key);
                if (inOutRecordVo == null) {
                    inOutRecordVo = new InOutRecordVo();
                    inOutRecordVo.setName(tInOutRecordBody.getName());
                    inOutRecordVo.setModel(tInOutRecordBody.getModel());
                    inOutRecordVo.setSpecification(specification);
                    inOutRecordVo.setType(tInOutRecordBody.getType());
                    inOutRecordVo.setInventoryQuantity(tInOutRecordBody.getQuantityInventory());
                    inOutRecordVo.setUnit(tInOutRecordBody.getUnit());
                    totalMap.put(key, inOutRecordVo);
                } else {
                    inOutRecordVo.setInventoryQuantity(NumberUtil.add(inOutRecordVo.getInventoryQuantity(), tInOutRecordBody.getQuantityInventory()));
                }
                // 根据材料编码 判断是材料还是半成品
                TMaterialInfo tMaterialInfo = tMaterialInfoMapper.selectTMaterialInfoByMaterialCode(model);
                String materialType = tMaterialInfo.getMaterialType();
                // 材料
                if("1".equals(materialType)) {
                    // 库存明细
                    TInventoryDetails inventoryDetails = new TInventoryDetails();
                    inventoryDetails.setName(name);
                    inventoryDetails.setModel(model);
                    inventoryDetails.setSpecification(specification);
                    inventoryDetails.setUnit(tInOutRecordBody.getUnit());
                    inventoryDetails.setInventoryQuantity(tInOutRecordBody.getQuantityInventory());
                    inventoryDetails.setInventoryType(Convert.toInt(tInOutRecord.getInventoryType()));
                    inventoryDetails.setInventoryDate(new Date());
                    inventoryDetails.setSupplierCode(tInOutRecordBody.getSupplierCode());
                    inventoryDetails.setRemark(tInOutRecordBody.getRemark());
                    String batchNo = redisCache.generateOrderNum();
                    inventoryDetails.setBatchNumber(batchNo);
                    inventoryDetailsList.add(inventoryDetails);
                } else if("2".equals(materialType)){
                    // 半成品
                    BigDecimal quantityInventory = tInOutRecordBody.getQuantityInventory();
                    Integer integer = Convert.toInt(quantityInventory);
                    for (int i = 0; i < integer; i++) {
                        TInventoryDetails inventoryDetails = new TInventoryDetails();
                        inventoryDetails.setName(name);
                        inventoryDetails.setModel(model);
                        inventoryDetails.setSpecification(specification);
                        inventoryDetails.setUnit(tInOutRecordBody.getUnit());
                        inventoryDetails.setInventoryQuantity(new BigDecimal(1));
                        inventoryDetails.setInventoryType(Convert.toInt(tInOutRecord.getInventoryType()));
                        inventoryDetails.setInventoryDate(new Date());
                        inventoryDetails.setSupplierCode(tInOutRecordBody.getSupplierCode());
                        inventoryDetails.setRemark(tInOutRecordBody.getRemark());
                        String batchNo = redisCache.generateOrderNum();
                        inventoryDetails.setBatchNumber(batchNo);
                        // app端半成品入库，不走如下逻辑
                        if( ("0".equals(tInOutRecord.getOperateType()) || "8".equals(tInOutRecord.getOperateType())) && !"1".equals(tInOutRecord.getIsCreateIdentifier()) ) {
                            // 采购入库和其他入库，自动生成半成品的唯一编号
                            String snowId = SnowIdUtils.getSnowId();
                            inventoryDetails.setIdentifier(snowId);
                            appProcessRecordService.createPurchaseBanChengPing(inventoryDetails);
                        }
                        inventoryDetailsList.add(inventoryDetails);
                    }
                }
            }
            //跑完之后开始组装库存明细，获取原来对应的库存明细，进行库存相加等操作
            for (String s : totalMap.keySet()) {
                TStorehouseDetails tStorehouseDetail = new TStorehouseDetails();
                InOutRecordVo inOutRecordVo = totalMap.get(s);
                // 库存类型;0-产品 1-材料
                tStorehouseDetail.setInventoryType(tInOutRecord.getInventoryType());
                // 分类 分类 0-采购入库 1-退料入库 2-生产入库 3-退货入库
                tStorehouseDetail.setType(tInOutRecord.getOperateType());
                // 单位 unit
                tStorehouseDetail.setUnit(inOutRecordVo.getUnit());
                // 库存数量
                tStorehouseDetail.setInventoryQuantity(inOutRecordVo.getInventoryQuantity());
                // 材料名称
                tStorehouseDetail.setName(inOutRecordVo.getName());
                // 材料编码
                tStorehouseDetail.setModel(inOutRecordVo.getModel());
                // 材料规格
                tStorehouseDetail.setSpecification(inOutRecordVo.getSpecification());
                tStorehouseDetails.add(tStorehouseDetail);
            }
        } else {
            throw new RuntimeException("入库清单不能为空！");
        }
        if (CollUtil.isNotEmpty(tStorehouseDetails)) {
            for (TStorehouseDetails tStorehouseDetail : tStorehouseDetails) {
                //查询条件为 仓库id+名称+型号+
                TStorehouseDetails tStorehouseDetails1 = tStorehouseDetailsMapper.selectOne(
                        new LambdaQueryWrapper<TStorehouseDetails>()
                                .eq(TStorehouseDetails::getName, tStorehouseDetail.getName())
                                .eq(TStorehouseDetails::getModel, tStorehouseDetail.getModel())
                                .eq(TStorehouseDetails::getSpecification, tStorehouseDetail.getSpecification())
                );
                if (tStorehouseDetails1 == null) {
                    tStorehouseDetailsMapper.insert(tStorehouseDetail);
                } else {
                    tStorehouseDetails1.setInventoryQuantity(NumberUtil.add(tStorehouseDetail.getInventoryQuantity(), tStorehouseDetails1.getInventoryQuantity()));
                    tStorehouseDetailsMapper.updateById(tStorehouseDetails1);
                }
            }
            // 插入库存明细
            if (CollUtil.isNotEmpty(inventoryDetailsList)) {
                tInventoryDetailsService.saveBatch(inventoryDetailsList);
            }
        }
        //若是采购入库，则采购入库完成后修改采购单的对应入库状态
        if ("0".equals(tInOutRecord.getOperateType())) {
            String associatedNo = tInOutRecord.getAssociatedNo();
            TMaterialPickingNew tMaterialPickingNew = tMaterialPickingNewMapper.selectOne(new LambdaQueryWrapper<TMaterialPickingNew>().eq(TMaterialPickingNew::getNo, associatedNo));
            if (tMaterialPickingNew != null) {
                tMaterialPickingNew.setProductStatus("1");
                tMaterialPickingNewMapper.updateById(tMaterialPickingNew);
            }
            // 更修材料供应商历史价格
            List<TMaterialSupplier> updateList = new ArrayList<>();
            List<TMaterialSupplier> insertList = new ArrayList<>();
            for (TInOutRecordBody tInOutRecordBody : tInOutRecord.getInOutRecordBodyList()) {
                //名称
                String name = tInOutRecordBody.getName();
                //规格
                String specification = tInOutRecordBody.getSpecification();
                //型号
                String model = tInOutRecordBody.getModel();
                // 供应商
                String supplierCode = tInOutRecordBody.getSupplierCode();
                BigDecimal unitPrice = tInOutRecordBody.getUnitPrice();
                List<TMaterialInfo> tMaterialInfos = tMaterialInfoMapper.selectList(Wrappers.lambdaQuery(TMaterialInfo.class)
                        .eq(TMaterialInfo::getMaterialCode, model)
                        .eq(TMaterialInfo::getMaterialName, name)
                        .eq(TMaterialInfo::getMaterialModel, specification)
                );
                List<Supplier> suppliers = supplierMapper.selectList(Wrappers.lambdaQuery(Supplier.class).eq(Supplier::getSupplierCode, supplierCode));
                if(CollUtil.isNotEmpty(tMaterialInfos) && CollUtil.isNotEmpty(suppliers)) {
                    TMaterialInfo tMaterialInfo = tMaterialInfos.get(0);
                    Long materialInfoId = tMaterialInfo.getId();
                    Supplier supplier = suppliers.get(0);
                    Long supplierId = supplier.getId();
                    LambdaQueryWrapper<TMaterialSupplier> qw = Wrappers.lambdaQuery(TMaterialSupplier.class);
                    qw.eq(TMaterialSupplier::getMaterialId, materialInfoId);
                    qw.eq(TMaterialSupplier::getSupplierId, supplierId);
                    qw.eq(TMaterialSupplier::getDelFlag, "0");
                    TMaterialSupplier tMaterialSupplier = tMaterialSupplierService.getOne(qw);
                    if(tMaterialSupplier != null) {
                        tMaterialSupplier.setPrice(unitPrice);
                        tMaterialSupplier.setUpdateTime(new Date());
                        tMaterialSupplier.setUpdateBy(SecurityUtils.getUsername());
                        updateList.add(tMaterialSupplier);
                    } else {
                        tMaterialSupplier = new TMaterialSupplier();
                        tMaterialSupplier.setMaterialId(materialInfoId);
                        tMaterialSupplier.setSupplierId(supplierId);
                        tMaterialSupplier.setMaterialSupplierCode(model + "-" + supplierCode);
                        tMaterialSupplier.setPrice(unitPrice);
                        tMaterialSupplier.setCreateBy(SecurityUtils.getUsername());
                        tMaterialSupplier.setCreateTime(new Date());
                        tMaterialSupplier.setUpdateTime(new Date());
                        tMaterialSupplier.setUpdateBy(SecurityUtils.getUsername());
                        insertList.add(tMaterialSupplier);
                    }
                }
            }
            if(CollUtil.isNotEmpty(updateList)) {
                tMaterialSupplierService.updateBatchById(updateList);
            }
            if(CollUtil.isNotEmpty(insertList)) {
                tMaterialSupplierService.saveBatch(insertList);
            }
        }

        //若是退料入库，需要修改该退料入库所选择的领料单下的材料可退料数量 returnableQuantity
        if ("1".equals(tInOutRecord.getOperateType())) {
            List<TInOutRecordBody> inOutRecordBodies = tInOutRecord.getInOutRecordBodyList();
            for (TInOutRecordBody body : inOutRecordBodies) {
                // 查询该 出入库记录对应的 领料单材料
                // 通过 no, productName, ComponentItemNumber, specification, unit 来唯一确定
                // 退料入库时， associatedNo 只有 tInOutRecord 中有，tInOutRecordBody 中没有
                LambdaQueryWrapper<TMaterialPickingListNew> queryWrapper = Wrappers
                        .lambdaQuery(TMaterialPickingListNew.class)
                        .eq(TMaterialPickingListNew::getNo, tInOutRecord.getAssociatedNo())
                        .eq(TMaterialPickingListNew::getProductName, body.getName())
                        .eq(TMaterialPickingListNew::getComponentItemNumber, body.getModel())
                        .eq(TMaterialPickingListNew::getSpecification, body.getSpecification())
                        .eq(TMaterialPickingListNew::getUnit, body.getUnit());
                TMaterialPickingListNew tMaterialPickingListNew = tMaterialPickingListNewMapper.selectOne(queryWrapper);

                // 更新可退料数量: 新的returnableQuantity = 旧的returnableQuantity - 本次入库数量 quantityInventory
                tMaterialPickingListNew.setReturnableQuantity(NumberUtil.sub(tMaterialPickingListNew.getReturnableQuantity(), body.getQuantityInventory()));

                // 更新
                tMaterialPickingListNewMapper.updateTMaterialPickingListNew(tMaterialPickingListNew);
            }
        }

        // 若是其他入库，且tInOutRecordBody 中的 remark 字段 和 identifier 字段内容不为空且相同时，添加人均产能记录 -- 入库
        //（app端入库操作TAppProcessRecordServiceImpl.inStorage() 会使tInOutRecordBody 中的 remark 字段和 identifier 字段相同且不为空）
        List<TInOutRecordBody> bodies = tInOutRecord.getInOutRecordBodyList();
        if ("8".equals(tInOutRecord.getOperateType())
                && bodies.get(0).getRemark() != null && bodies.get(0).getIdentifier() != null
                && bodies.get(0).getRemark().equals(bodies.get(0).getIdentifier())) {
            // 从 tInOutRecord 中获取入库时间，并将时间切分为年/月/日
            Date warehousingTime = tInOutRecord.getWarehousingTime();
            SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd");
            String transformDate = format.format(warehousingTime);
            String year = transformDate.substring(0, 4);
            String month = transformDate.substring(5, 7);
            String day = transformDate.substring(8, 10);

            // 通过 tInOutRecord 获取 人均产能记录需要的 userName 和 jobNumber 字段
            // 从 tInOutRecord 中获取 applicant (applicant 对应的是 SysUser 的 nickName)
            // 并通过 nickName 获取 jobNumber
            String userName = tInOutRecord.getApplicant();
            String jobNumber = userService.selectUserByNickName(userName).getJobNumber();

            // 遍历 bodies 插入人均产能记录
            for(TInOutRecordBody body : bodies) {
                // 插入人均产能记录 -- 入库
                TCapacityUser capacityUser = new TCapacityUser();
                capacityUser.setYear(year);
                capacityUser.setMonth(month);
                capacityUser.setDay(day);
                // 入库的工艺名称统一规定为 入库
                capacityUser.setProcessName("入库");
                capacityUser.setUserName(userName);
                capacityUser.setJobNumber(jobNumber);
                capacityUser.setQuantity(body.getQuantityInventory());
                capacityUserService.insertTCapacityUser(capacityUser);
            }
        }
    }

    /**
     * @param documentType 类型 （订单，领料，采购等）
     * @param count        返回生成的集合数量
     * @param length       后面的序号是几位序号 如 length = 2 -> 01  length = 4 ->0001
     *                     通过单证类型创建批号
     * @return
     */
    @Override
    public List<String> createBatchNumberByDocumentType(String documentType, int count, Integer length) {
        List<String> result = new ArrayList<>();
        switch (documentType) {
            //订单
            case "DD":
                return createBatchNumber(count, length, "t_orders_new", "order_number", 14, 2, 10, 11, 14, "DD");
            case "SD":
                return createBatchNumber(count, length, "t_production_order", "production_order_code", 14, 2, 10, 11, 14, "SD");
            //领料单
            case "LL":
                return createBatchNumber(count, length, "t_material_picking_new", "no", 14, 2, 10, 11, 14, "LL");
            // 领料单明细批号
            case "PH":
                return createBatchNumber(count, length, "t_material_picking_list_new", "batch_number", 14, 2, 10, 11, 14, "PH");
            //采购单
            case "CG":
                return createBatchNumber(count, length, "t_purchase_order_header", "ticket_no", 14, 2, 10, 11, 14, "CG");
            //工单
            case "GD":
                return createBatchNumber(count, length, "t_worksheets", "work_number", 14, 2, 10, 11, 14, "GD");
            //出入库
            case "inOutRecord":
                return createBatchNumber(count, length, "t_storehouse_details", "batch_number", 12, 0, 8, 9, 12, "");
            case "inOutRecordNo":
                return createBatchNumber(count, length, "t_in_out_record", "single_number", 12, 0, 8, 9, 12, "");
            case "WT":
                return createBatchNumber(count, length, "t_work_task", "task_no", 14, 2, 10, 11, 14, "WT");
        }
        return result;
    }

    /**
     * 创建批号
     */
    @Override
    public List<String> createBatchNumber(int count, int digits, String tableName, String columnName, Integer lengths, int fromIndex, int toIndex, int lastFromIndex, int lastToIndex, String prefix) {
        List<String> result = new ArrayList<>();
        //判断是否为空，或者位数不为10位，则序号从01开始
        String nowDateTime = DateUtils.dateTime();
        //先获取原来的循环到那个批号了
        String maxBatchNumber = tStorehouseDetailsMapper.getMaxBatchNumber(tableName, columnName, prefix + nowDateTime);

        if (maxBatchNumber == null || lengths != (maxBatchNumber.length())) {
            for (int i = 0; i < count; i++) {
                BigDecimal add = NumberUtil.add(new BigDecimal(0), i + 1);
                result.add(prefix + nowDateTime + getNumber(add.intValue(), digits));
            }
        } else {
            String sub = StrUtil.sub(maxBatchNumber, fromIndex, toIndex);
            String subLast = StrUtil.sub(maxBatchNumber, lastFromIndex, lastToIndex);
            if (sub.equals(nowDateTime)) {
                for (int i = 0; i < count; i++) {
                    BigDecimal add = NumberUtil.add(new BigDecimal(subLast), i + 1);
                    result.add(prefix + sub + getNumber(add.intValue(), digits));
                }
            } else {
                for (int i = 0; i < count; i++) {
                    BigDecimal add = NumberUtil.add(new BigDecimal(0), i + 1);
                    result.add(prefix + nowDateTime + getNumber(add.intValue(), digits));
                }
            }
        }
        return result;
    }

    @Override
    public List<TPurchaseSingleDetailVo> getTPurchaseSingleDetailByTicketNo(String ticketNo) {
        List<TPurchaseSingleDetailVo> tPurchaseSingleDetailByTicketNo = tInOutRecordMapper.getTPurchaseSingleDetailByTicketNo(ticketNo);
        return tPurchaseSingleDetailByTicketNo.stream().filter(vo -> vo.getLaveQuantityInventoryed().compareTo(vo.getQuantityInventory()) < 0).collect(Collectors.toList());
    }
    @Override
    public List<TPurchaseSingleDetailVo> getTProductSingleDetailByTicketNo(String ticketNo) {
        List<TPurchaseSingleDetailVo> tPurchaseSingleDetailByTicketNo = tInOutRecordMapper.getTProductSingleDetailByTicketNo(ticketNo);
        return tPurchaseSingleDetailByTicketNo.stream().filter(vo -> vo.getLaveQuantityInventoryed().compareTo(vo.getQuantityInventory()) < 0).collect(Collectors.toList());
    }

    /**
     * @param i 位数 若为4 则  0001
     * @param s 输入值
     *          获取对应位数的序号
     * @return
     */
    private static String getNumber(int s, int i) {
        NumberFormat formatter = NumberFormat.getNumberInstance();
        formatter.setMinimumIntegerDigits(i);
        formatter.setGroupingUsed(false);
        return formatter.format(s);
    }

    /**
     * 修改出入库记录
     *
     * @param tInOutRecord 出入库记录
     * @return 结果
     */
    @Override
    public int updateTInOutRecord(TInOutRecord tInOutRecord) {
        getBasis(tInOutRecord, types.UPDATE);
        return tInOutRecordMapper.updateTInOutRecord(tInOutRecord);
    }

    /**
     * 批量删除出入库记录
     *
     * @param ids 需要删除的出入库记录ID
     * @return 结果
     */
    @Override
    public int deleteTInOutRecordByIds(Long[] ids) {
        return tInOutRecordMapper.deleteTInOutRecordByIds(ids);
    }

    @Override
    public void getBasis(TInOutRecord tInOutRecord, types inputType) {
        switch (inputType) {
            case ADD:
                tInOutRecord.setCreateTime(DateUtils.getNowDate());
                tInOutRecord.setCreateBy(SecurityUtils.getLoginUser().getUser().getUserId() + "");
                break;
            case UPDATE:
                tInOutRecord.setUpdateTime(DateUtils.getNowDate());
                tInOutRecord.setUpdateBy(SecurityUtils.getLoginUser().getUser().getUserId() + "");
                break;
            default:
                break;
        }
    }

    /**
     * 根据transportCode修改出库单的状态
     */
    @Override
    public int updateOutboundOrderStatus(String outboundOrderStatus, String transportCode) {
        // 从货物对象 获取出库单号
        TGoods tGoods = tGoodsMapper.selectOne(
                new LambdaQueryWrapper<TGoods>()
                        .eq(TGoods::getTransportCode, transportCode)
                        .eq(TGoods::getDelFlag, "0")
        );
        if (Objects.nonNull(tGoods)) {
            String outRecordNo = tGoods.getOutRecordNo();
            // 根据出库单号，将出库修改为 未出库
            LambdaUpdateWrapper<TInOutRecord> uw = new LambdaUpdateWrapper<>();
            uw.set(TInOutRecord::getOutboundOrderStatus, outboundOrderStatus).eq(TInOutRecord::getSingleNumber, outRecordNo);
            update(uw);
        }
        return 1;
    }


    @Override
    public TInOutRecordBody getDetailByWorkShees(String workNumber) {
        return tInOutRecordMapper.getDetailByWorkShees(workNumber);
    }

    @Transactional(rollbackFor = Exception.class)
    public int invalidate(Long id) {
        TInOutRecord byId = getById(id);
        byId.setOutboundOrderStatus("3");
        updateById(byId);
        List<TInOutRecordBody> tInOutRecordBodies = tInOutRecordBodyMapper.selectList(
                Wrappers.lambdaQuery(TInOutRecordBody.class)
                        .eq(TInOutRecordBody::getTInOutRecordId, id)
                        .eq(TInOutRecordBody::getDelFlag, "0")
        );
        for (TInOutRecordBody tInOutRecordBody : tInOutRecordBodies) {
            tProductInformationMapper.updateOutRecordFLag(tInOutRecordBody.getIdentifier(), "0");
            ordersNewListService.subOrAddOutQuantity(tInOutRecordBody, false);
        }
        return 1;
    }

    @Override
    public void llOut(String userName, String remark,String singleNumber) {
        tProductInformationMapper.llOut(userName,remark,singleNumber);
    }
}
~~~

