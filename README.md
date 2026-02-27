# 🛸 Alien Alien 

## 🏆 인하대학교 미래인재개발원 게임잼 1등 수상작
![Main Capsule](https://github.com/user-attachments/assets/7b966522-e941-4ec8-807e-7c58e415a900)

### <p align="center">🎬 [플레이 영상 보러가기](https://www.youtube.com/watch?v=J4nrQGVUj44&list=PLBVctyobs3SpTdc_aWn4Tk0Tt6t2tkpxi)</p>


&nbsp; &nbsp; 

## 📝 프로젝트 소개
- **게임 장르** : 2D 플랫포머 슈팅 게임
- **제작 기간** : 2025.3Q (2일)
- **게임 요약** : 화성에서 끊임없이 몰려오는 외계인을 피해 무사히 탈출해야 하는 긴박한 슈팅 액션 게임입니다.


&nbsp; &nbsp; 

## 🎯 프로젝트 목표
- **협업 및 소통**: 2인 팀으로 2일간 긴밀하게 소통하며 단기 프로젝트 내 효율적인 협업 프로세스 경험
- **역량 발휘**: 학습한 개발 기술을 총동원하여 실제 플레이 가능한 수준의 게임 사이클 구축
- **완성도 있는 제작**: 제한된 시간 내에 기획부터 배포까지 마침표를 찍는 '완결성 있는 개발' 지향

&nbsp; &nbsp; 

## 🛠 기술 경험 (Tech Experience)

### 🐙 GitHub & 협업 워크플로우
이번 프로젝트를 통해 **첫 GitHub 협업**을 경험하며 코드 관리의 중요성을 학습했습니다.

- **GitHub Desktop 활용**: GUI 기반 툴을 사용하여 복잡한 Git 명령어 대신 직관적인 버전 관리 및 커밋 로그 관리 수행
- **Branch & Merge 전략**: 기능을 분담하여 각자의 브랜치에서 개발 후, 작업 완료 시 `Main` 브랜치에 안전하게 `Merge`하여 통합
- **실시간 동기화 (Pull & Push)**: 작업 시작 전 `Pull`을 통해 팀원의 최신 코드를 반영하고, 내 작업물을 `Push`하며 충돌(Conflict)을 최소화하는 협업 사이클 숙달

&nbsp; &nbsp; 
  
## 🛠 핵심 개발 상세 (Technical Deep Dive)

**1. 행동의 인터페이스화 (인터페이스 및 구현체 정리)** 

단기간 내 다양한 몬스터 패턴을 구현하기 위해 행동 단위(Behavior)를 인터페이스로 추상화하고, 이를 조립하는 **전략 패턴(Strategy Pattern)** 을 설계했습니다.

감지(IPlayerDetector), 추적(IChaseBehavior), 공격(IAttackBehavior) 등을 독립된 인터페이스로 분리하여 몬스터마다 최적화된 구현체를 장착할 수 있도록 구성했습니다.

![image17](https://github.com/user-attachments/assets/95fc3e8d-563e-49ed-8e80-68f7501c9b27)
![image18](https://github.com/user-attachments/assets/cf006020-91fa-4996-b4a5-2924f03ce69a)


<details>
<summary>👾 몬스터 AI: 독립 인터페이스 기반 행동 설계</summary>

<blockquote>
<details>
<summary>🔍 감지 로직 (IPlayerDetector)</summary>
<blockquote>원형, 수평, 반원 등 상황에 맞는 감지 방식을 선택하여 적용합니다.</blockquote>
  
![image32](https://github.com/user-attachments/assets/3fc1d565-ca1c-483b-a196-0270e9c33802)

```cs
// 대표 예시: HorizontalRangeDetector
public class HorizontalRangeDetector : IPlayerDetector
{
    private float rangeX;
    private float rangeY;
    private Transform monsterTransform;
    public HorizontalRangeDetector(float rangeX, Transform monsterTransform, float rangeY = 1.0f)
    {
        this.rangeX = rangeX;
        this.rangeY = rangeY;
        this.monsterTransform = monsterTransform;
    }

    public bool IsPlayerDetected(Transform monster, Transform player)
    {
        float dx = player.position.x - monster.position.x;
        float dy = Mathf.Abs(player.position.y - monster.position.y);
        bool playerInFront =
            (dx > 0 && monsterTransform.localScale.x > 0) || // 오른쪽 바라보고 오른쪽에 있음
            (dx < 0 && monsterTransform.localScale.x < 0);   // 왼쪽 바라보고 왼쪽에 있음
        
        return Mathf.Abs(dx) < rangeX && dy < rangeY && playerInFront;
    }
}
// 이외 구현체: CircleDetector, UpperHalfCircleDetector
```
</details>

<details>
<summary>👁️ 경계 로직 (IWatchBehavior)</summary>
<blockquote>플레이어 발견 시 즉시 추격하지 않고 대치하는 상태를 구현합니다.</blockquote>
  
![GameZam 2026-02-27 21-16-19](https://github.com/user-attachments/assets/05ddc13d-1e50-4e34-973e-1b68dfd17cd8)

```cs
public class PassiveWatch : IWatchBehavior
{
    private float chaseRange;
    private float watchStartTime = -1f;
    private bool isWatching = false;
    public PassiveWatch(float chaseRange)
    {
        this.chaseRange = chaseRange;
    }

    public void Watch(Transform monster, Transform player, Rigidbody2D rb)
    {
        if (!isWatching)
        {
            watchStartTime = Time.time;
            isWatching = true;
        }
        // 바라보기만 하고 안 움직임
        if (player.position.x < monster.position.x && monster.localScale.x > 0)
            monster.localScale = new Vector3(-1, 1, 1);
        else if (player.position.x > monster.position.x && monster.localScale.x < 0)
            monster.localScale = new Vector3(1, 1, 1);


        rb.velocity = Vector2.zero;
    }

    public bool ShouldChase(Transform monster, Transform player)
    {
        return isWatching && Time.time - watchStartTime >= 1f && Mathf.Abs(player.position.x - monster.position.x) < chaseRange;
    }

    public void ResetWatch()
    {
        isWatching = false;
        watchStartTime = -1f;
    }
}
```
</details>

<details>
<summary>🏃 추격 로직 (IChaseBehavior)</summary>
<blockquote>몬스터의 이동 방식(지상/비행)에 따른 추격 알고리즘입니다.</blockquote>

```cs
// 대표 예시: MaintainDistanceChase
public class MaintainDistanceChase : IChaseBehavior
{
    private float speed;
    private float stopDistance;

    public MaintainDistanceChase(float speed, float stopDistance)
    {
        this.speed = speed;
        this.stopDistance = stopDistance;
    }

    public void Chase(Transform monster, Transform player, Rigidbody2D rb)
    {
        float distance = Vector2.Distance(monster.position, player.position);

        if (distance > stopDistance)
        {
            Vector2 direction = (player.position - monster.position).normalized;
            rb.velocity = new Vector2(direction.x * speed, rb.velocity.y);
        }
        else
        {
            rb.velocity = Vector2.zero;
        }
    }
}
// 이외 구현체: FullChase
```
</details>

</details>


**2. 최적화 및 공통 시스템 (Optimization & Core Systems)**

**"성능 최적화와 코드 재사용성 극대화"** 를 목표로 설계된 시스템입니다. 빈번한 객체 생성/파괴로 인한 가비지 컬렉션(GC) 부하를 방지하고, 인터페이스를 통해 다양한 개체들이 동일한 로직으로 상호작용할 수 있도록 구현했습니다.

성능 최적화: 슈팅 게임 특성상 대량으로 발생하는 투사체(Bullet)를 Object Pooling 기법으로 관리하여 프레임 드랍을 방지했습니다.

유지보수성: IHealth 인터페이스를 도입하여 플레이어, 일반 몬스터, 보스가 각기 다른 클래스임에도 불구하고 데미지 처리 로직을 단일화했습니다.

![GameZam 2026-02-27 23-05-56](https://github.com/user-attachments/assets/38e59cae-4360-4f72-ace8-ed3ece65270b)


<details>
<summary>🚀 최적화 및 공통 시스템</summary>

<blockquote>
<details>
<summary>💣 오브젝트 풀링 (ObjectPool)</summary>
<blockquote>MonsterBullet, BossBullet 등 대량의 투사체 생성 시 발생하는 부하를 방지합니다.</blockquote>

```cs
// MonsterBullet.cs

public void Init(ObjectPool<MonsterBullet> objectPool)
{
    this.objectPool = objectPool;
}

public void Fire(Vector2 dir)
{
    isReleased = false;
    rb.velocity = dir.normalized * speed;
    StartCoroutine(ReturnBullet());
}

public void Deactivate()
{
    StopAllCoroutines();
    rb.velocity = Vector2.zero;
    objectPool.Release(this);
}

IEnumerator ReturnBullet()
{
    yield return bulletLifeTime;
    rb.velocity = Vector2.zero;
    objectPool.Release(this);
}

private void OnTriggerEnter2D(Collider2D collision)
{
    // 플레이어만 맞추게 처리
    if (collision.CompareTag("Player"))
    {
        // 데미지 처리
        if (collision.TryGetComponent(out IHealth health))
        {
            health.TakeDamage(damage);
        }

        rb.velocity = Vector2.zero;
        if (!isReleased)
        {
            isReleased = true;
            objectPool.Release(this);
        }
    }
    // 벽, 바닥 등 기타 모든 물리 오브젝트에도 반응
    else if (collision.gameObject.layer == LayerMask.NameToLayer("Ground"))
    {
        rb.velocity = Vector2.zero;
        if (!isReleased)
        {
            isReleased = true;
            objectPool.Release(this);
        }
    }
    // 그 외엔 무시 (적끼리 충돌 X, 다른 총알 등)
}
// 리턴 시 objectPool.Release(this) 호출
```
</details>

<details>
<summary>🏥 체력 및 데미지 시스템 (IHealth)</summary>
<blockquote>인터페이스를 통해 플레이어와 몬스터의 데미지 처리를 일원화했습니다.</blockquote>

```cs
// MonsterStats.cs
public void TakeDamage(int damage)
{
    // Calculate effective damage after defense
    int effectiveDamage = Mathf.Max(damage - defense, 1);   // Ensure at least 1 damage is dealt
    currentHp -= effectiveDamage;
    currentHp = Mathf.Max(currentHp, 0);

    if (healthBar != null)
        healthBar.SetHealth(currentHp);

    // Check if the monster is dead
    if (currentHp <= 0)
    {
        Die();
    }
}
```
</details>
</details>

**3. 시스템 매니저 (Singleton & Managers)**

**"전역적인 데이터 관리와 일관된 게임 흐름 제어"** 를 위해 핵심 매니저들을 구축했습니다. 제네릭 싱글톤을 활용하여 매니저 클래스 생성 시 발생하는 중복 코드를 제거하고, 사운드 및 씬 전환 로직을 중앙 집중화했습니다.

아키텍처: Singleton<T> 제네릭 클래스를 상속받는 구조를 채택하여, 새로운 매니저가 추가되더라도 안전하고 빠르게 싱글톤 패턴을 적용할 수 있습니다.

리소스 관리: SoundManager를 통해 BGM과 효과음을 분리하여 관리하며, 씬 로드 시 자동으로 해당 스테이지에 맞는 배경음악이 재생되도록 자동화했습니다.

<details>
<summary>🎵 시스템 매니저 (Singleton)</summary>

<blockquote>
<details>
<summary>👑 제네릭 싱글톤 (Singleton.cs)</summary>
<blockquote>코드 재사용성을 높이기 위한 제네릭 기반 싱글톤 구조입니다.</blockquote>

```cs

public class Singleton<T> : MonoBehaviour where T : MonoBehaviour
{
    static T instance;
    
    public static T Instance
    {
        get
        {
            if (instance == null)
            {
                instance = (T)FindObjectOfType(typeof(T));

                if(instance == null)
                {
                    //Debug.LogError(typeof(T) + " 싱글톤이 씬에 없습니다! 씬에 싱글톤을 배치하세요.");
                }
            }
            return instance;
        }
    }

    private void Awake()
    {
        if(instance != null && instance.gameObject != this.gameObject)
        {
            Destroy(gameObject);
            return;
        }

        OnAwakeWork();
    }

    private void OnDestroy()
    {
        OnDestroyedWork();
    }

    protected virtual void OnAwakeWork() { }

    protected virtual void OnDestroyedWork() { }
}
```
</details>

<details>
<summary>🔊 사운드 관리 (SoundManager.cs)</summary>
<blockquote>딕셔너리와 큐를 활용한 효율적인 사운드 재생 시스템입니다.</blockquote>

```cs
public class SoundManager : Singleton<SoundManager>
{
    //public static SoundManager Instance { get; private set; }

    //사운드 매핑
    [SerializeField] private List<SoundEntry> soundTable;

    private Dictionary<SoundType, AudioClip> clipMap;
    private Queue<AudioClip> playQueue = new Queue<AudioClip>();

    private AudioSource audioSource;
    private AudioSource bgmSource;


    protected override void OnAwakeWork()
    {
        DontDestroyOnLoad(gameObject);

        // AudioSource 컴포넌트 세팅
        audioSource = gameObject.AddComponent<AudioSource>();
        audioSource.playOnAwake = false;
        audioSource.loop = false;
        audioSource.spatialBlend = 0;

        // 배경음악용 AudioSource 설정
        bgmSource = gameObject.AddComponent<AudioSource>();
        bgmSource.playOnAwake = false; 
        bgmSource.loop = true;
        bgmSource.spatialBlend = 0;
        bgmSource.volume = 0.1f; // 배경음악 볼륨 설정 (필요시 조정)

        // Dictionary로 매핑 초기화
        clipMap = new Dictionary<SoundType, AudioClip>();
        foreach (var entry in soundTable)
            clipMap[entry.type] = entry.clip;

        // 씬 전환 이벤트 구독
        SceneManager.sceneLoaded += OnSceneLoaded;
        // 첫 씬(Startup)도 바로 재생
        OnSceneLoaded(SceneManager.GetActiveScene(), LoadSceneMode.Single);

    }

    protected override void OnDestroyedWork()
    {
        // 반드시 해제
        SceneManager.sceneLoaded -= OnSceneLoaded;
    }

    // 씬이 바뀔 때마다 호출
    private void OnSceneLoaded(Scene scene, LoadSceneMode mode)
    {
        // 이전 BGM 멈춤
        if (bgmSource.isPlaying) bgmSource.Stop();

        // 재생할 타입 결정
        SoundType typeToPlay;
        if (scene.name == "Main") typeToPlay = SoundType.MainBackgroundMusic;
        else if (scene.name == "Stage") typeToPlay = SoundType.BackgroundMusic;
        else if (scene.name == "Death")
        {
            typeToPlay = SoundType.GameOver;
            bgmSource.loop = false;
        }
        else if (scene.name == "Clear") typeToPlay = SoundType.GameClear;
        else return; // 그 외 씬은 BGM 없음

        // 클립이 있으면 재생
        if (clipMap.TryGetValue(typeToPlay, out var clip) && clip != null)
        {
            bgmSource.clip = clip;
            bgmSource.Play();
        }
    }

    public void PlaySound(SoundType type, float volume = 0.3f)
    {
        if (type == SoundType.BackgroundMusic || type == SoundType.MainBackgroundMusic)
            return; // 배경음악은 별도로 처리

        if (clipMap.TryGetValue(type, out var clip) && clip != null)
        {
            playQueue.Enqueue(clip);

            var next = playQueue.Dequeue();
            audioSource.PlayOneShot(next, volume);
        }
        else
        {
            Debug.LogWarning($"SoundManager: '{type}' 클립이 없습니다.");
        }
    }
```
</details>
</details>



---------
-------
-------
----------
----------
