# Permission
Library, BaseActivty 사용한 권한 설정

### BaseActivty로 사용
```java
// 추상 클래스로 MainActivty에서 상속받도록 한다
public abstract class BaseActivity extends AppCompatActivity {

    // 체크할 퍼미션
    private static String[] permissions = {Manifest.permission.READ_EXTERNAL_STORAGE};
    public static int REQ_CODE = 999;

    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        checkVersion();

    }

    // 모든 작업이 끝나면 init()을 호출하고, MainActivity에서는 onCreate()대신 모든 초기화 작업을 init()에서 해준다
    abstract void init();

    private void checkVersion(){
        // 버전이 마시멜로 미만인 경우는 패스
        if(Build.VERSION.SDK_INT < Build.VERSION_CODES.M){
            init();
            // 이상인 경우는 일단 허용이 된 퍼미션이 무엇인지 체크한다.
        } else {
            checkAlreadyGrantedPermission();
        }
    }

    /**
     * 이미 체크된 퍼미션이 있는지 확인하고, 체크되지 않았다면 시스템에 onRequestPermission()으로 권한을 요청한다.
     */
    @RequiresApi(api = Build.VERSION_CODES.M)
    private void checkAlreadyGrantedPermission() {
        boolean isAllGranted = true;
        for(String perm : permissions){
            // 만약 원하는 퍼미션이 하나라도 허용이 안 되었다면 false로 전환
            if(checkSelfPermission(perm) != PackageManager.PERMISSION_GRANTED){
                isAllGranted = false;
            }
        }
        // 만약 전부 허용이 되었다면 다음 액티비티로 넘어간다.
        if(isAllGranted){
            init();
            // 허용되지 않는 것이 있다면 시스템에 권한신청한다.
        } else {
            requestPermissions(permissions, REQ_CODE);
        }
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        onResult(grantResults);
    }

    /**
     * 시스템 권한체크가 끝난 후 호출
     */
    private void onResult(int[] grantResults){
        boolean isAllGranted = true;
        for(int granted : grantResults){
            if(granted != PackageManager.PERMISSION_GRANTED){
                isAllGranted = false;
            }
        }
        // 허용되면 init()으로 원하는 함수를 실행하고
        if(isAllGranted){
            init();
            // 허용되지 않는 것이 있다면 시스템에 권한신청한다.
        } else {
            finish();
        }
    }
}
```

### Libarary로 사용

#### Permission 영역
```java
// 체크할 퍼미션
private static String[] permissions = {Manifest.permission.ACCESS_FINE_LOCATION, Manifest.permission.ACCESS_COARSE_LOCATION};
public static int REQ_CODE = 999;
private volatile static CheckPermission permission;

/**
 * 데이터베이스 연결이 아니고, 가장 먼저 하는 일이기 때문에 싱글턴으로 해도 메모리 누수가 생기지 않는다.
 */
public static CheckPermission getInstance(){
    if(permission == null){
        permission = new CheckPermission();
    }
    return permission;
}
```
```java
public void checkVersion(Activity activity){
    // 버전이 마시멜로 미만인 경우는 패스
    if(Build.VERSION.SDK_INT < Build.VERSION_CODES.M){
        ((MainActivity)activity).init();
        // 이상인 경우는 일단 허용이 된 퍼미션이 무엇인지 체크한다.
    } else {
        checkAlreadyGrantedPermission(activity);
    }
}

/**
 * 이미 체크된 퍼미션이 있는지 확인하고, 체크되지 않았다면 시스템에 onRequestPermission()으로 권한을 요청한다.
 */
@RequiresApi(api = Build.VERSION_CODES.M)
private void checkAlreadyGrantedPermission(Activity activity) {
    boolean isAllGranted = true;
    for(String perm : permissions){
        // 만약 원하는 퍼미션이 하나라도 허용이 안 되었다면 false로 전환
        if(activity.checkSelfPermission(perm) != PackageManager.PERMISSION_GRANTED){
            isAllGranted = false;
        }
    }
    // 만약 전부 허용이 되었다면 다음 액티비티로 넘어간다.
    if(isAllGranted){
        ((MainActivity)activity).init();
        // 허용되지 않는 것이 있다면 시스템에 권한신청한다.
    } else {
        activity.requestPermissions(permissions, REQ_CODE);
    }
}
```

```java
/**
 * 시스템 권한체크가 끝난 후 호출
 */
public void onResult(int[] grantResults, Activity activity){
    boolean isAllGranted = true;
    for(int granted : grantResults){
        if(granted != PackageManager.PERMISSION_GRANTED){
            isAllGranted = false;
        }
    }
    // 허용되면 init()으로 원하는 함수를 실행하고
    if(isAllGranted){
        ((MainActivity)activity).init();
        // 허용되지 않는 것이 있다면 시스템에 권한신청한다.
    } else {

    }
}

/**
 * MainActivity가 스스로를 넘겨주면, 이곳에서 MainActivity 를 대신해 메소드를 호출해주는 콜백 메서드
 */
public interface CallBack {
    void init();
}
```

#### MainActivty 영역

```java
/**
 * 시스템에게 권한 요청하면 시스템이 인자로 결과 값들을 넘겨준다.
 * @param requestCode   REQ_CODE
 * @param permissions   시스템에게 요청한 권한들
 * @param grantResults  허용됬는지 여부가 담겨온다
 */
@Override
public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
    CheckPermission.getInstance().onResult(grantResults, this);
}
```
