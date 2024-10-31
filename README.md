#Q2.

별도의 ui스크립트를 만드는 이유
단일 책임 원칙 - 각각 하나만을 담당
확장하기에 편하고 수정하기에도 편하기 때문
능동적인 개념을 필요로 하기때문

인터페이스는 다중상속을 지원하며 클래스가 구현해야 하는 함수를 정하는 것이다.

데미지를 받을 수 있는 객체에 IDamagable이라는 인터페이스에 데미지를 받는 함수를 넣고 상속시켜
확장성을 확보하고 코드를 조금 더 깔끔하게 만들 수 있도록 한다
public interface IDamagable
{
    void TakePhysicalDamage(int damageAmount);
}


    public void TakePhysicalDamage(int damageAmount)
    {
        health.Subtract(damageAmount);
        onTakeDamage?.Invoke();
    }


핵심 로직을 분석해보세요. (UI 스크립트 구조
Condition => 
hunger, stamina, health 최댓값, 최소값, 더함과 뺌의 제한을 검 / 각각에 적용

uicondition =>
condition을 한번에 관리하여 확장성 확보
ex) 동료, 몬스터

PlayerCondition
그값을 player에게 적용


CampFire
IDamageable 인터페이스를 구현한 객체의 목록을 만들어서 충돌한 오브젝트에서 그 인터페이스를 감지 후 있다면 목록에 넣어 
DealDamage를 실행 시 데미지를 주도록 함 / 빠져나가면 목록에서 제거


DamageIndicator
코루틴을 활용하여 OnTakeDamage 이벤트에 flash 함수 구독
이를 통해 데미지를 입으면 일정시간동안 화면을 빨갛게 만듦# SteadyStudy
 

 #Q3.
 public interface IInteractable
{
    public string GetInteractPrompt(); 
    public void OnInteract();
}
인터페이스로 상호작용 가능한 것들에 상속시킬 것이므로 상호작용할때의 함수와 상호작용시 얻을 프롬포트를 스트링값으로 받을 수 있게 하였다.

public class Interaction : MonoBehaviour
{
    public float checkRate = 0.05f;  체크 주기 - update문에 넣으면 너무 자주 실행되기에 주기를 정해준다.
    private float lastCheckTime;   체크 주기를 맞추기위해 체크타임을 설정한다.
    public float maxCheckDistance;   체크거리
    public LayerMask layerMask;   레이어마스크

    public GameObject curInteractGameObject;   현재 상호작용중인 오브젝트
    private IInteractable curInteractable;      위의 인터페이스를 상속시킨 변수 선언

    public TextMeshProUGUI promptText;   유니티에 프롬포트를 넣기위해 만듦
    private Camera camera;    카메라를 기준으로 잡기위해 선언

    void Start()
    {
        camera = Camera.main;       카메라가 메인임을 선언
    }

    void Update()
    {
        if(Time.time - lastCheckTime > checkRate)   체크주기를 맞추기 위해 최근의 체크시간에서 시간을 빼 체크주기를 넘었다면 실행
        {
            lastCheckTime = Time.time;  실행되었으니 초기화

            Ray ray = camera.ScreenPointToRay(new Vector3(Screen.width / 2, Screen.height / 2));   레이를 발사할때 화면의 정중앙에 발사하기에 화면의
            RaycastHit hit;    레이케스트 히트를 선언 높이와 넓이를 2로 나누어 값을 구하여 정중앙을 맞춤
            if(Physics.Raycast(ray, out hit, maxCheckDistance, layerMask))  
            레이케스트를 통해 레이를 발사, 히트 - 정보를 담는 객체, 최대 체크거리, 레이어마스크
            {   즉, 레이가 충돌한다면 실행
                if(hit.collider.gameObject != curInteractGameObject)  
                 충돌한 객체가 현재 상호작용중인 게임오브젝트가 아니면 실행      
                {
                    curInteractGameObject = hit.collider.gameObject;        
                    현재 상호작용중인 오브젝트를 충돌한 게임오브젝트와 동일시하고
                    curInteractable = hit.collider.GetComponent<IInteractable>(); 
                    충돌한 오브젝트에서 IInteractable인터페이스 컴포넌트를 가져와 curInteractable과 맞추고
                    SetPromptText();                                                            SetPromptText함수 실행
                }
            }
            else    레이가 충돌하지 않았다면                                       
            {
                curInteractGameObject = null;  curInteractGameObject를 널로  
                curInteractable = null;   curInteractable 널로 
                promptText.gameObject.SetActive(false); 프롬포트 게임오브젝트를 비활성화시킨다.
            }
        }
    }

    private void SetPromptText()
    {
        promptText.gameObject.SetActive(true);  게임오브젝트의 프롬포트 텍스트를 활성화
        promptText.text = curInteractable.GetInteractPrompt();  프롬포트 텍스트를 가져온다
    }

    public void OnInteractInput(InputAction.CallbackContext context)
    {     상호작용키를 누르고 현재 상호작용을 하려는 물체가 인터페이스가 있다면 실행
        if(context.phase == InputActionPhase.Started && curInteractable != null)
        {
            curInteractable.OnInteract();   인터페이스의 함수 실행
            curInteractGameObject = null;   상호작용 중인 게임오브젝트를 널로
            curInteractable = null;  받아온 인터페이스도 널로
            promptText.gameObject.SetActive(false);  게임 오브젝트의 프롬포트 비활성화
        }
    }
}
