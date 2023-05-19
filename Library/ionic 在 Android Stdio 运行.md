```shell
npx cap add android
npm run build
npx cap copy
npx cap sync
npx cap open android

ionic capacitor run android -l --external
```

问题：

`Caused by: org.gradle.api.internal.plugins.PluginApplicationException: Failed to apply plugin [id ‘com.android.application’]`

解决办法：

在 gradle.properties 添加 `android.overridePathCheck=true` 即可

