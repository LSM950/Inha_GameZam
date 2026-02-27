# 🛸 Alien Alien 

## 🏆 인하대학교 미래인재개발원 게임잼 1등 수상작
![Main Capsule](https://github.com/user-attachments/assets/7b966522-e941-4ec8-807e-7c58e415a900)

### <p align="center">🎬 [플레이 영상 보러가기](https://www.youtube.com/watch?v=J4nrQGVUj44&list=PLBVctyobs3SpTdc_aWn4Tk0Tt6t2tkpxi)</p>

---

## 📝 프로젝트 소개
- **게임 장르** : 2D 플랫포머 슈팅 게임
- **제작 기간** : 202X.XX.XX ~ 202X.XX.XX (2일)
- **게임 요약** : 화성에서 끊임없이 몰려오는 외계인을 피해 무사히 탈출해야 하는 긴박한 슈팅 액션 게임입니다.

---

## 🎯 프로젝트 목표
- **협업 및 소통**: 2인 팀으로 2일간 긴밀하게 소통하며 단기 프로젝트 내 효율적인 협업 프로세스 경험
- **역량 발휘**: 학습한 개발 기술을 총동원하여 실제 플레이 가능한 수준의 게임 사이클 구축
- **완성도 있는 제작**: 제한된 시간 내에 기획부터 배포까지 마침표를 찍는 '완결성 있는 개발' 지향

---

## 🛠 기술 경험 (Tech Experience)

### 🐙 GitHub & 협업 워크플로우
이번 프로젝트를 통해 **첫 GitHub 협업**을 경험하며 코드 관리의 중요성을 학습했습니다.

- **GitHub Desktop 활용**: GUI 기반 툴을 사용하여 복잡한 Git 명령어 대신 직관적인 버전 관리 및 커밋 로그 관리 수행
- **Branch & Merge 전략**: 기능을 분담하여 각자의 브랜치에서 개발 후, 작업 완료 시 `Main` 브랜치에 안전하게 `Merge`하여 통합
- **실시간 동기화 (Pull & Push)**: 작업 시작 전 `Pull`을 통해 팀원의 최신 코드를 반영하고, 내 작업물을 `Push`하며 충돌(Conflict)을 최소화하는 협업 사이클 숙달
  
## 🛠 핵심 개발 상세 (Technical Deep Dive)

**1. 행동의 인터페이스화 (인터페이스 및 구현체 정리)** 

단기간 내 다양한 몬스터 패턴을 구현하기 위해 행동 단위(Behavior)를 인터페이스로 추상화하고, 이를 조립하는 **전략 패턴(Strategy Pattern)**을 설계했습니다.

감지(IPlayerDetector), 추적(IChaseBehavior), 공격(IAttackBehavior) 등을 독립된 인터페이스로 분리하여 몬스터마다 최적화된 구현체를 장착할 수 있도록 구성했습니다.

<details>
<summary>👾 몬스터 AI: 독립 인터페이스 기반 행동 설계</summary>

<blockquote>
<details>
<summary>🔍 감지 로직 (IPlayerDetector)</summary>
<blockquote>원형, 수평, 반원 등 상황에 맞는 감지 방식을 선택하여 적용합니다.</blockquote>

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


2. 효율적인 전투 및 최적화 시스템
이 부분은 성능과 관리 편의성을 강조하며 클래스 이름을 배치합니다.

<details>
<summary>🚀 최적화 및 공통 시스템</summary>

<blockquote>
<details>
<summary>💣 오브젝트 풀링 (ObjectPool)</summary>
<blockquote>MonsterBullet, BossBullet 등 대량의 투사체 생성 시 발생하는 부하를 방지합니다.</blockquote>

```cs
// MonsterBullet.cs 내 구현 예시
public void Init(ObjectPool<MonsterBullet> objectPool)
{
    this.objectPool = objectPool;
}
// 리턴 시 objectPool.Release(this) 호출
```
</details>

<details>
<summary>🏥 체력 및 데미지 시스템 (IHealth)</summary>
<blockquote>인터페이스를 통해 플레이어와 몬스터의 데미지 처리를 일원화했습니다.</blockquote>

```cs
// MonsterStats.cs
public class MonsterStats : MonoBehaviour, IHealth
{
    public void TakeDamage(int damage) { /* 데미지 및 사망 처리 */ }
}
```
</details>
</details>

<details>
<summary>🎵 시스템 매니저 (Singleton)</summary>


<details>
<summary>👑 제네릭 싱글톤 (Singleton.cs)</summary>
<blockquote>코드 재사용성을 높이기 위한 제네릭 기반 싱글톤 구조입니다.</blockquote>

```cs
public class Singleton<T> : MonoBehaviour where T : MonoBehaviour
```
</details>

<details>
<summary>🔊 사운드 관리 (SoundManager.cs)</summary>
<blockquote>딕셔너리와 큐를 활용한 효율적인 사운드 재생 시스템입니다.</blockquote>

```cs
public class SoundManager : Singleton<SoundManager>
{
    // SoundType에 따른 오디오 클립 자동 매핑 및 재생
}
```
</details>
</details>



---------
-------
-------
----------
----------
