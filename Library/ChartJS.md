## Y 坐标轴设定数值范围

```typescript
let chart = new Chart(ctx, {
    type: 'line',
    data: {
        datasets: [{
            label: 'First dataset',
            data: [0, 20, 40, 50]
        }],
        labels: ['January', 'February', 'March', 'April']
    },
    options: {
        scales: {
            yAxes: [{	//设置 Y 轴
                ticks: {
                    suggestedMin: 50,
                    suggestedMax: 100
                }
            }]
        }
    }
});
```