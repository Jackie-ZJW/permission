QuickPermission
README: 中文 | English

Introduction to the QuickPermission
This is a convenient library for permission management in Android, which makes application permission and business code logic be separated easily, and does not care about permission application and callback.

The original
How did you manage Android permissions in the past?
Determine whether you permission first, then apply for permission, finally onRequestPermissionsResult check one by one as a result, to execute the business logic?
There is a call authority, many places to use, you have to write every call authority judgment application and result processing?
Now here's the good news:
There's a library for permissions management in Android where you tell it what permissions you need and then tell it what you want to execute.


Integration method
 Step 1. Add the Jcenter repository to your build file Add it in your root build.gradle at the end of repositories:

allprojects {
		repositories {
			...
			jcenter()
		}
	}

Step 2. Add the dependency

dependencies {
	        implementation 'com.jackiezjw.quickpermission:quickpermission:0.0.2'
	}

Step 3. call onRequestPermissionsResult(). Going to use QuickPermission onRequestPermissionsResult in Activity, call QuickPermissionHelper. GetInstance (). OnRequestPermissionsResult can;
If you have BaseActivity, you only need to set it up once in BaseActivity.

 @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        //Inject the callback using QuickPermissionHelper
        QuickPermissionHelper.getInstance().onRequestPermissionsResult(requestCode, permissions, grantResults, this);
    }


Function using
Now let's see how to use it.

1.To check the permission
All you need to do is call the hasPermission method of QuickPermission, which allows multiple permissions to be passed in at the same time.

QuickPermission.build().hasPermission(this, Manifest.permission.CALL_PHONE);

2.To apply for permission
If you need to apply for permissions when your application starts, and you don't care about the result of permissions,
You only need to call the requestPermission method of QuickPermission, which supports multiple permissions passing in.

 QuickPermission.build().requestPermission(MainActivity.this, Manifest.permission.CALL_PHONE);

3.Results that require permissions
If you need to know the result of the user's selection after applying permission, while executing your own method myVoid(),
So just do it in onPermissionsAccess,
In onPermissionsDismiss is the user rejecting permission feedback.

 QuickPermission.build()
                .mRequestCode(RC_CODE_CALLPHONE)
                .mContext(NeedReslutActivity.this)
                .mPerms(Manifest.permission.CALL_PHONE)
                .mResult(new QuickPermissionResult() {
                    @Override
                    public void onPermissionsAccess(int requestCode) {
                        super.onPermissionsAccess(requestCode);
                        //Do what you want
                    }

                    @Override
                    public void onPermissionsDismiss(int requestCode, @NonNull List<String> permissions) {
                        super.onPermissionsDismiss(requestCode, permissions);
                        //What do you do if your access is denied by the user
                    }
                }).requestPermission();

4.Sometimes the user rejects the permission, and the popup request is forbidden, what should I do?
Same as above, so long as in onDismissAsk, you can get the banned result, while you want to notice that onDismissAsk returns false by default,
So if you modify return true yourself, you're going to look at it as having handled the ban result, you're not going to call back the onPermissionsDismiss method,
Call the openAppDetails method, you can popover to guide the user to set the permissions in the interface, and check the results in the onActivityResult.

 QuickPermission quickPermission = QuickPermission.build()
                .mRequestCode(RC_CODE_CALLPHONE)
                .mContext(DismissAskActivity.this)
                .mPerms(Manifest.permission.CALL_PHONE)
                .mResult(new QuickPermissionResult() {
                    @Override
                    public void onPermissionsAccess(int requestCode) {
                        super.onPermissionsAccess(requestCode);
                        //Do what you want
                    }

                    @Override
                    public void onPermissionsDismiss(int requestCode, @NonNull List<String> permissions) {
                        super.onPermissionsDismiss(requestCode, permissions);
                        //What do you do if your access is denied by the user
                    }

                    @Override
                    public boolean onDismissAsk(int requestCode, @NonNull List<String> permissions) {
                        //What do you do if your permissions are blocked by the user and cannot be requested
                        quickPermission.openAppDetails(DismissAskActivity.this, "Call Phone - Give me the permission to dial the number for you");
                        return true;
                    }
                });
        quickPermission.requestPermission();


        @Override
            protected void onActivityResult(int requestCode, int resultCode, Intent data) {
                super.onActivityResult(requestCode, resultCode, data);
                if (QuickPermission.APP_SETTINGS_RC == requestCode) {
                    //Setting interface return
                    //Result from system setting
                    if (quickPermission.hasPermission(DismissAskActivity.this)) {
                        //Do what you want
                    } else {
                        //It still doesn't give you permission to go back from Settings
                    }

                }
            }