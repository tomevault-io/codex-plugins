# Test - vJass Game Project

게임 개발을 위한 vJass 언어 기반 프로젝트입니다. 캐릭터, 보스, UI, 게임 시스템 등 다양한 게임 요소를 모듈식으로 구성하고 있습니다.

## 📁 프로젝트 구조

### 🎮 핵심 모듈

#### **Boss/** - 보스 AI 및 전투 시스템
- `Boss1-1.j` ~ `Boss1-4.j`: 보스 1의 단계별 전투 AI
- `BossAggro.j`: 보스 어그로(적대도) 관리 시스템

#### **Hero/** - 캐릭터 및 스킬 시스템
- `Potion.j`: 포션 관리 및 소비 시스템
- `SkillButton.j`: 스킬 버튼 UI 관리
- `SkillDash.j`: 대시 스킬 구현
- **캐릭터별 폴더**: Bandi, Chen, Jack, Lucia, Momizi, Narmaya (각 캐릭터의 고유 능력 및 스킬)

#### **Data/** - 게임 데이터 및 설정
- `Data Unit.j`: 유닛(캐릭터, 몬스터) 데이터 정의
- `Data Item.j`: 아이템 데이터 및 속성
- `Data Buff.j`: 버프/디버프 데이터
- `Data Arcana.j`: 아르카나(특수 능력) 데이터
- `Data Map.j`: 맵 관련 설정
- `Native.j`: 네이티브 함수 정의

#### **UI/** - 사용자 인터페이스
- `UI_HP.j`: 체력 표시 바
- `UI_Skill.j`: 스킬 UI
- `UI_Item.j`: 인벤토리 UI
- `UI_Map.j`: 맵 미니맵 UI
- `UI_Quest.j`: 퀘스트 UI
- `UI_Shop.j`: 상점 UI
- `UI_Msg.j`: 메시지/채팅 UI
- `UI_BossHP.j`: 보스 체력 표시
- `UI_FPS.j`: FPS 표시
- `UI_Achievement.j`: 업적 시스템
- 기타 UI 모듈들

#### **System/** - 게임 시스템
- `AOEHit.j`: 범위 공격 시스템
- `Cooldown.j`: 스킬 쿨다운 관리
- `DamageEffect.j`: 일반 피해 효과
- `DamageEffectBoss.j`: 보스 전용 피해 효과
- `ItemPickUp.j`: 아이템 습득 시스템
- `Shield.j`: 보호막 시스템
- `StatsSetting.j`: 능력치 설정
- `SaveLoad.j`: 게임 저장/로드
- `Market.j`: 마켓 시스템
- `Daily.j`: 일일 시스템

#### **Library/** - 재사용 가능한 라이브러리
- `Knockback.j`: 넉백 효과
- `Missile.j`: 미사일 시스템
- `Splash.j`: 스플래시 효과
- `Stun.j`: 스턴 상태 관리
- `UnitIndexer.j`: 유닛 인덱싱
- `Tick.j`: 틱 기반 시간 관리
- `TimerUtils.j`: 타이머 유틸리티
- `Sort.j`: 정렬 알고리즘
- `ArrayEx.j`: 배열 확장

#### **import/** - 외부 API 및 프레임워크
- `JNCommon.j`: 공통 함수 모음
- `JNMemory.j`: 메모리 관리
- `JNString.j`: 문자열 처리
- `JAPI*.j`: JAPI 플러그인 모음
- `DzAPI*.j`: DzAPI 플러그인 모음

#### **vJDK/** - vJDK 프레임워크
게임 플러그인 및 애드온 시스템

---

## 🛠️ 함수 작성 가이드

### 1. 기본 함수 구조

```vjass
// 함수 선언
function FunctionName takes unit u, integer value returns nothing
    // 함수 본문
endfunction

// 리턴값이 있는 함수
function GetDamage takes unit attacker, unit target returns real
    local real damage = 10.0
    return damage
endfunction
```

### 2. 모듈별 함수 작성 가이드

#### **System/ 시스템 함수**
- 전역 게임 시스템 함수들
- 명명 규칙: `System_*` 또는 `Init*`
- 예: `function InitDamageSystem takes nothing returns nothing`

#### **Library/ 라이브러리 함수**
- 재사용 가능한 유틸리티 함수
- 명명 규칙: `Lib_*` 또는 모듈명으로 시작
- 예: `function Knockback_Apply takes unit u, real distance returns nothing`

#### **Hero/ 캐릭터 함수**
- 캐릭터별 스킬 및 능력
- 명명 규칙: `Character_SkillName` 또는 `SkillName_*`
- 예: `function Lucia_DashSkill takes unit caster returns nothing`

#### **Boss/ 보스 함수**
- 보스 AI 및 공격 패턴
- 명명 규칙: `Boss_*` 또는 `BossAction_*`
- 예: `function Boss1_AttackPattern takes unit boss returns nothing`

#### **UI/ UI 함수**
- 화면 표시 및 업데이트
- 명명 규칙: `UI_*`
- 예: `function UI_UpdateHP takes unit u, real hp, real maxhp returns nothing`

### 3. 일반적인 작성 패턴

#### 초기화 함수
```vjass
function InitSystem takes nothing returns nothing
    // 초기 설정
    set udg_GlobalVariable = 0
    // 타이머 시작
    call TimerStart(CreateTimer(), 0.03, true, function OnTick)
endfunction
```

#### 주기적 업데이트 함수
```vjass
function OnTick takes nothing returns nothing
    local unit u = null
    // 모든 유닛 순회
    // 로직 처리
endfunction
```

#### 이벤트 처리 함수
```vjass
function OnUnitDamaged takes nothing returns nothing
    local unit damaged = GetEventDamageSource()
    local real damage = GetEventDamage()
    // 피해 처리 로직
endfunction
```

### 4. 변수 명명 규칙

```vjass
// 전역 변수 (udg_로 시작)
globals
    unit array udg_HeroUnit
    real udg_GlobalCooldown = 0.0
    integer udg_PlayerCount = 0
endglobals

// 지역 변수 (명확한 이름)
local unit caster = null
local real damage = 10.0
local integer count = 0
```

### 5. ⚠️ 변수 할당 - SET 키워드 필수

**vJass에서 변수를 할당할 때는 반드시 `set` 키워드를 사용해야 합니다.**

#### ✅ 올바른 사용법

```vjass
// 지역 변수 할당
local unit u = null
set u = GetSpellAbilityUnit()

// 전역 변수 할당
set udg_GlobalCooldown = 10.0

// 구조체 멤버 할당
set fx.caster = u
set fx.BHPxN = 100

// 배열 요소 할당
set BHPBar[0] = DzCreateFrameByTagName(...)
set heroes[i] = GetNthPlayerUnitSimple(i + 1, i)

// 조건문 내 할당
if isAlive then
    set hp = 100
else
    set hp = 0
endif

// 조건부 함수 결과 할당
set p = Player(fx.playerId)
set isLocalPlayer = GetLocalPlayer() == p
set texturePath = "war3mapImported\\ZT-[BOSS]-B" + I2S(fx.BHPxL) + ".blp"
```

#### ❌ 잘못된 사용법

```vjass
// ❌ set 키워드 빠뜨림
local unit u = null
u = GetSpellAbilityUnit()  // 오류!

// ❌ 전역 변수 할당 시 set 미사용
udg_GlobalCooldown = 10.0  // 오류!

// ❌ 구조체 할당 시 set 미사용
fx.caster = u  // 오류!

// ❌ 조건문 내에서도 필수
if isAlive then
    hp = 100  // 오류! set 필수
endif

// ❌ 함수 결과도 set으로 할당
p = Player(fx.playerId)  // 오류!
```

#### 📋 체크리스트

변수 할당 시 다음을 확인하세요:
- [ ] 변수 할당 앞에 `set` 키워드가 있는가?
- [ ] 지역 변수, 전역 변수, 구조체 멤버 모두 `set` 사용?
- [ ] 배열 요소 할당도 `set` 사용?
- [ ] 조건문 내 할당에도 `set` 사용?
- [ ] 함수 반환값 할당에도 `set` 사용?

---

### 6. 메모리 관리

```jass
// 변수 초기화
local unit u = null

// 사용 후 정리
set u = null  // 참조 제거

// 배열 초기화
local unit array heroes
local integer i = 0
loop
    set heroes[i] = null
    set i = i + 1
    exitwhen i > 5
endloop
```

### 7. 조건문 및 루프

```vjass
// if-then-else
if GetUnitState(u, UNIT_STATE_LIFE) > 0 then
    call SetUnitState(u, UNIT_STATE_LIFE, 100)
else
    call UnitRemoveAbility(u, ABILITY_CODE)
endif

// loop
local integer i = 0
loop
    set udg_HeroUnit[i] = GetNthPlayerUnitSimple(i + 1, i)
    set i = i + 1
    exitwhen i >= 6
endloop
```

### 7. 스킬/능력 함수 템플릿

```vjass
function Skill_Activate takes nothing returns nothing
    local unit caster = GetSpellAbilityUnit()
    local unit target = GetSpellTargetUnit()
    local real castX = GetSpellTargetX()
    local real castY = GetSpellTargetY()
    local integer level = GetUnitAbilityLevel(caster, ABILITY_CODE)
    
    // 스킬 로직
    
    set caster = null
    set target = null
endfunction

function Skill_Loop takes nothing returns nothing
    // 주기적 효과 처리
endfunction
```

### 8. 데미지 계산 함수 템플릿

```vjass
function CalculateDamage takes unit attacker, unit target, integer skillLevel returns real
    local real baseDamage = 10.0
    local real multiplier = 1.0 + (skillLevel * 0.5)
    local real finalDamage = baseDamage * multiplier
    
    return finalDamage
endfunction

function ApplyDamage takes unit attacker, unit target, real damage returns nothing
    local real newHP = GetUnitState(target, UNIT_STATE_LIFE) - damage
    
    if newHP <= 0 then
        call UnitRemoveAbility(target, 'A000')
    else
        call SetUnitState(target, UNIT_STATE_LIFE, newHP)
    endif
endfunction
```

### 9. 모범 사례

✅ **해야 할 것:**
- 함수 끝에 지역 변수 정리
- 명확한 함수명과 변수명 사용
- 매직 넘버 대신 상수 정의
- 주석으로 복잡한 로직 설명
- 일관된 들여쓰기 사용

❌ **하지 말아야 할 것:**
- 변수 초기화 없이 사용
- 과도하게 깊은 중첩 루프
- 메모리 누수 (변수 정리 누락)
- 전역 변수 과다 사용
- 마법 숫자 하드코딩

---

## 🚀 주요 기능

- **캐릭터 시스템**: 다중 캐릭터 관리 및 개별 능력 정의
- **보스 전투**: 다단계 보스 AI 및 전투 시스템
- **스킬 및 능력**: 캐릭터별 고유 스킬과 버프/디버프 시스템
- **아이템 및 인벤토리**: 아이템 획득, 인벤토리 관리, 상점 시스템
- **UI 시스템**: 포괄적인 사용자 인터페이스
- **게임 시스템**: 쿨다운, 데미지, 범위 공격, 보호막, 저장/로드 등

## 🛠️ 기술 스택

- **언어**: vJass (Warcraft III 확장 스크립팅 언어)
- **프레임워크**: vJDK, CommonAPI
- **라이브러리**: 메모리 관리, 타이머, 유틸리티 모음

## 📝 시작하기

1. 프로젝트 폴더 구조 이해하기
2. `import/` 폴더의 기본 라이브러리 숙지
3. 해당 모듈(Hero, Boss, System 등)에서 함수 작성
4. 함수 작성 가이드에 따라 일관된 코드 유지

## 📌 주의사항

- vJDK 및 JAPI 라이브러리는 외부 종속성입니다
- 메모리 관리는 신중하게 처리해야 합니다
- 각 모듈의 의존성을 확인한 후 수정하세요
- 함수는 모듈별로 명확히 분류하여 작성하세요

## ⚙️ 일시정지 상태 유닛의 방향 변경

**vJass에서 일시정지된 유닛(`PauseUnit(unit, true)`)의 방향을 변경해야 할 때 주의사항:**

### 🔧 올바른 방법

```vjass
// 1단계: 방향 설정
call SetUnitFacing(caster, angle)
call EXSetUnitFacing(caster, angle)

// 2단계: 위치 갱신 (필수!)
// 방향 변경을 즉시 적용하려면 위치를 다시 설정해야 함
call SetUnitPosition(caster, GetWidgetX(caster), GetWidgetY(caster))
```

### ✅ 중요 포인트

- **일시정지 상태에서도 방향 설정 가능**: SetUnitFacing(), EXSetUnitFacing(), SetUnitPosition() 순서대로 호출시 정상 작동

### ❌ 잘못된 방법

```vjass
// 방법 1: 위치 갱신 없이 방향만 설정 (효과 없음)
call SetUnitFacing(caster, angle)
call EXSetUnitFacing(caster, angle)

// 방법 2: 일시정지 해제 (일시 정지를 해제하면 안됨)
call PauseUnit(caster, false)
call SetUnitFacing(caster, angle)
call EXSetUnitFacing(caster, angle)
call PauseUnit(caster, true)
```

## 🔄 버전 정보

- **저장소**: https://github.com/zhongpo95/Test
- **분기**: main
- **언어**: vJass
- **최종 수정**: 2025년 12월

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zhongpo95)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/zhongpo95)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
