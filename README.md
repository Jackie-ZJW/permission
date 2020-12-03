QuickPermission
README: 中文 | English

QuickPermission简介
这个是一个方便Android中权限管理的库，它使得申请权限和业务代码逻辑简单分离，不去关心权限的申请和回调。

初衷
以前你是怎么管理Android的权限的？
先判断有没有权限，再申请权限，最后onRequestPermissionsResult中检查一个个结果，再执行自己的业务逻辑？
有一个打电话的权限，好多地方都要用，你每次使用打电话都要写一遍权限的判断申请和结果处理？
那现在有一个好消息：
这儿有一个方便Android中权限管理的库，你告诉它你需要的权限，然后再告诉它你想要执行什么，就可以了。


集成方法
 第一步. 添加Jcenter 将其添加到根build.gradle中.目前Android Studio默认就会添加Jcenter，如果没用添加的话就自己手动添加下。

allprojects {
		repositories {
			...
			jcenter()
		}
	}

第二步. 添加依赖

dependencies {
	        implementation 'com.jackiezjw.quickpermission:quickpermission:0.0.2'
	}

第三步. 调用 onRequestPermissionsResult().
   在将要使用QuickPermission的Activity中重写onRequestPermissionsResult方法,并在该方法中调用QuickPermissionHelper.getInstance().onRequestPermissionsResult即可；
如果你有BaseActivity，那么只需要在BaseActivity中设置一次即可。

 @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        //使用QuickPermissionHelper注入回调
        QuickPermissionHelper.getInstance().onRequestPermissionsResult(requestCode, permissions, grantResults, this);
    }


功能使用
接下来看下怎么使用。

1.检测权限
只需要调用QuickPermission的hasPermission方法,支持多个权限同时传入。

QuickPermission.build().hasPermission(this, Manifest.permission.CALL_PHONE);

2.申请权限
如果你在应用启动时需要申请权限，而且并不关注权限的结果，
只需要调用QuickPermission的requestPermission方法，支持多个权限传入。

 QuickPermission.build().requestPermission(MainActivity.this, Manifest.permission.CALL_PHONE);

3.需要权限的结果
如果你需要知道申请权限后用户的选择结果，同时去执行自己的方法myVoid(),
那么在onPermissionsAccess中去做已经成功获取到权限后需要做的事情就可以了，在onPermissionsDismiss是用户拒绝了权限的反馈。

 QuickPermission.build()
                .mRequestCode(RC_CODE_CALLPHONE)
                .mContext(NeedReslutActivity.this)
                .mPerms(Manifest.permission.CALL_PHONE)
                .mResult(new QuickPermissionResult() {
                    @Override
                    public void onPermissionsAccess(int requestCode) {
                        super.onPermissionsAccess(requestCode);
                        //做你想做的
                    }

                    @Override
                    public void onPermissionsDismiss(int requestCode, @NonNull List<String> permissions) {
                        super.onPermissionsDismiss(requestCode, permissions);
                        //你的权限被用户拒绝了你怎么办
                    }
                }).requestPermission();

4.有时用户拒绝了权限，而且禁止了弹出询问，我该怎么办？
同上，只要在onDismissAsk中，就可以得到被禁止的结果，同时你要注意onDismissAsk默认返回false，
如果你自己修改return true，将视为已经处理了禁止结果，将不再回调onPermissionsDismiss这个方法，
调用openAppDetails方法，可以弹窗引导用户去设置界面设置权限，在onActivityResult中检查结果。

 QuickPermission quickPermission = QuickPermission.build()
                .mRequestCode(RC_CODE_CALLPHONE)
                .mContext(DismissAskActivity.this)
                .mPerms(Manifest.permission.CALL_PHONE)
                .mResult(new QuickPermissionResult() {
                    @Override
                    public void onPermissionsAccess(int requestCode) {
                        super.onPermissionsAccess(requestCode);
                        //做你想做的
                    }

                    @Override
                    public void onPermissionsDismiss(int requestCode, @NonNull List<String> permissions) {
                        super.onPermissionsDismiss(requestCode, permissions);
                        //你的权限被用户拒绝了你怎么办
                    }

                    @Override
                    public boolean onDismissAsk(int requestCode, @NonNull List<String> permissions) {
                        //你的权限被用户禁止了并且不能请求了你怎么办
                        quickPermission.openAppDetails(DismissAskActivity.this, "Call Phone - Give me the permission to dial the number for you");
                        return true;
                    }
                });
        quickPermission.requestPermission();

        
        @Override
            protected void onActivityResult(int requestCode, int resultCode, Intent data) {
                super.onActivityResult(requestCode, resultCode, data);
                if (QuickPermission.APP_SETTINGS_RC == requestCode) {
                    //设置界面返回
                    //Result from system setting
                    if (quickPermission.hasPermission(DismissAskActivity.this)) {
                        //做你想做的
                    } else {
                        //从设置回来还是没给你权限
                    }
        
                }
            }