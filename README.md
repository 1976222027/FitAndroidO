# FitAndroid7

一行代码完成Android 7 FileProvider适配~

适配FileProvide需要声明provider，编写xml，以及在代码中做版本适配等...

可以抽取一个小库简化这些重复性操作，避免重复声明provider，编写xml，以及在代码中做版本适配...
fork  自 hangyangAndroid
适配androidx 增加AndroidO 安装apk 未知来源 权限

## 使用

```
compile 'com.mhy.base:fileprovider:1.0.0'
```

通过FileProvider7这个类完成uri的获取即可，例如：

* FileProvider7.getUriForFile
* FileProvider7.setIntentDataAndType
* FileProvider7.setIntentData


### 示例一 拍照

```java
private static final int REQUEST_CODE_TAKE_PHOTO = 0x110;
private String mCurrentPhotoPath;

public void takePhotoNoCompress(View view) {
    Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
    if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
        String filename = new SimpleDateFormat("yyyyMMdd-HHmmss", Locale.CHINA)
                .format(new Date()) + ".png";
        File file = new File(Environment.getExternalStorageDirectory(), filename);
        mCurrentPhotoPath = file.getAbsolutePath();
	     // 仅需改变这一行
        Uri fileUri = FileProvider7.getUriForFile(this, file);

        takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT, fileUri);
        startActivityForResult(takePictureIntent, REQUEST_CODE_TAKE_PHOTO);
    }
}

@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);
    if (resultCode == RESULT_OK && requestCode == REQUEST_CODE_TAKE_PHOTO) {
        mIvPhoto.setImageBitmap(BitmapFactory.decodeFile(mCurrentPhotoPath));
    }
    // else tip?

}
```

### 示例二 安装apk

```java
//检android O 安装未知来源
    private static final int INSTALL_PACKAGES_REQUESTCODE = 10011;
    private static final int GET_UNKNOWN_APP_SOURCES = 10012;

    private void checkAndroidO() {//有读写权限情况下 判断 未知应用
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) { //系统 Android O及以上版本
            //是否需要处理未知应用来源权限。 true为用户信任安装包安装 false 则需要获取授权
            boolean canRequestPackageInstalls = getPackageManager().canRequestPackageInstalls();
            if (canRequestPackageInstalls) {
                installApk();
            } else {
                //请求安装未知应用来源的权限
                ActivityCompat.requestPermissions(this, new String[]{Manifest.permission.REQUEST_INSTALL_PACKAGES}, INSTALL_PACKAGES_REQUESTCODE);
            }
        } else {  //直接安装流程
            installApk();
        }
    }

public void installApk(View view) {
    File file = new File(Environment.getExternalStorageDirectory(),
            "testandroid7-debug.apk");
    Intent intent = new Intent(Intent.ACTION_VIEW);
    // 仅需改变这一行
    FileProvider7.setIntentDataAndType(this,
            intent, "application/vnd.android.package-archive", file, true);
    startActivity(intent);
}
```


