# 03 - 설명: 기본 Nix 구문 및 flake.nix 입력

[<- 02 - 기본 리포지토리 설정](./02-basic-repository-setup_ko.md) | [04 - 설명: flake.nix 출력 함수 ->](./04-explain-outputs-function_ko.md)

첫 번째 홈 매니저 Flake 설정을 완료하고 저와 함께해 주신 것에 대한 보상으로, 이제 실제로 돌아가서 모든 것을 설명해 드리겠습니다! 맥락에 맞는 작동 예제를 갖는 것이 중요합니다.

개괄적인 설명을 드리려고 합니다. 토끼굴은 여전히 깊으니, 더 많은 것을 익히기 전까지는 여기서부터 얼마나 깊이 파고들지 조심하십시오. 이 작은 파일들에서 설명할 것이 얼마나 많은지 보는 것만으로도 천천히 진행해야 한다는 것을 안심시켜 줄 것입니다.

관련 문서나 기타 정보에 대한 링크를 추가할 것입니다. 살펴보는 것은 좋지만, **길을 잃지 않도록 주의하십시오**. 나중에 Nix에 더 익숙해지면 다시 방문할 수 있도록 저장해 두십시오.

_이 섹션들을 처음 읽을 때 이해하지 못해도 괜찮습니다. 나중에 더 많이 수정하면서 참고용으로 다시 돌아오십시오. 여기서 중요한 점은 이러한 것들이 나중에 참조할 개념으로 존재한다는 것입니다._

## Flake란 무엇입니까?

"Flake"라는 용어는 Nix의 눈송이 로고에서 따온 것입니다. 그 이상의 깊은 의미는 없습니다. 누군가 알려줄 때까지 한동안 궁금해했습니다.

기본적으로 ["Flake"](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake.html)는 Nix에서 완전히 독립적인 무언가를 정의하는 방법입니다. 모든 입력은 `inputs` 속성에 명시적으로 정의되며, 특정 Git 커밋이나 어떤 종류의 해시에 고정됩니다. 일반적인 Nix에서는 이러한 모든 해시를 직접 선언해야 하므로 고통스럽습니다. 이것이 Flake가 "실험적"으로 표시되었음에도 불구하고 인기 있는 이유 중 하나입니다.

Flake의 출력에는 여러 가지가 포함될 수 있습니다. 우리의 특정 경우에는 사용자 공간 환경을 정의하는 데 `home-manager`가 사용하는 출력을 정의하고 있습니다. 다른 경우에는 패키지, NixOS의 전체 시스템 구성 또는 단일 함수나 함수 라이브러리와 같은 것을 정의할 수 있습니다. 지금은 홈 매니저 구성을 만들고 있다는 점만 신경 쓰면 됩니다. 나중에는 특정 명령이 다른 출력 속성을 찾을 수 있습니다.

Flake를 빌드하거나 사용하면 `flake.lock` 파일을 참조하려고 합니다. 존재하지 않으면 정의된 입력의 최신 버전을 가져와 잠금 파일을 생성합니다. 입력이 추가되면 `flake.lock` 파일에 자동으로 저장됩니다. 입력이 제거되면 `flake.lock`에서 자동으로 제거됩니다. 재현성을 유지하기 위해 자동으로 업데이트되지는 않지만 [Nix 명령으로 명시적으로 업데이트](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake-update.html)할 수 있습니다.

## Flake의 구조

우리가 만든 Flake를 한 섹션씩 살펴보겠습니다.

### 중괄호

예, 중괄호입니다. 거기서부터 시작하겠습니다.

```nix
{
  # 왜 이것이 중괄호 안에 있습니까?
}
```

Nix에서 모든 `*.nix` 파일은 단일 [_Nix 표현식_](https://nixos.org/manual/nix/stable/language/)입니다. 이 경우 [_Nix 속성 집합_](https://nixos.org/manual/nix/stable/language/values.html#attribute-set)을 정의하고 있습니다. Nix 속성 집합은 기본적으로 JSON 덩어리와 같은 사전 개체입니다. 여러 속성을 포함하며, 각 속성은 여러 표현식 값을 보유합니다.

속성 집합은 중괄호로 정의됩니다. 속성 이름은 영숫자 값일 수 있으며 대시, 밑줄 또는 아포스트로피를 포함할 수 있습니다. 특정 구성을 위해 자체적으로 속성 집합을 보유하여 깊이 중첩되는 경우가 많습니다.

이것들은 절대적으로 어디에나 있습니다. Nix의 핵심입니다.

### Flake의 최상위 속성

Flake에는 우리가 신경 써야 할 몇 가지 최상위 속성이 있습니다. 구체적으로 `description`, `inputs`, `outputs`입니다.

`description`은 Flake를 설명하는 문자열입니다. 선택 사항입니다.

`inputs`는 Flake가 사용하는 입력 집합입니다. Flake는 `inputs`에 있는 것과 로컬 Git 리포지토리에 명시적으로 추가된 것만 봅니다.

`outputs`는 정의된 모든 `inputs`를 가져와 우리가 관심 있는 일부 구성을 생성하는 함수입니다.

### 설명

```nix
{
  description = "내 Home Manager 구성";
}
```

여기에는 문자열 리터럴을 보유하는 `description`이라는 속성이 있습니다. 가장 간단한 형태입니다. 세미콜론으로 끝나야 한다는 점에 유의하십시오. Nix의 속성 집합에 있는 모든 할당은 세미콜론으로 끝나야 합니다. 즉, 공백은 그다지 중요하지 않습니다.

```nix
{
  # 이것도 괜찮습니다!
  description =
    "내 Home Manager 구성";
}
```

이것은 어디에 사용됩니까? `nix flake info`를 실행하여 볼 수 있지만 솔직히 선택 사항입니다. 간단한 문자열 값 할당을 보여주는 편리한 방법이기 때문에 여기에 포함시킨 것뿐입니다! 실제로 루트 [flake.nix](./flake.nix) 파일을 보면 `description` 속성이 전혀 정의되어 있지 않은 것을 볼 수 있습니다. 공유 가능한 패키지에서 더 유용하므로 홈 매니저 설정에서는 원한다면 생략할 수 있습니다.

### 입력

이제 좀 더 흥미로운 것으로 넘어가겠습니다.

```nix
{
  inputs = {
    nixpkgs.url = "nixpkgs/nixos-23.11";

    home-manager = {
      url = "github:nix-community/home-manager/release-23.11";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };
}
```

이제 `inputs`가 Nix 속성 집합으로 설정되고 있음을 인식해야 합니다. `inputs` 내부에는 `nixpkgs`와 `home-manager`라는 두 가지 속성이 있습니다.

그런데 잠깐, 이 `nixpkgs.url`이라는 것은 무엇입니까?

```nix
{
  # 아래와 정확히 동일합니다.
  nixpkgs = {
    url = "nixpkgs/nixos-23.11";
  };

  # 완전히 동일합니다!
  nixpkgs.url = "nixpkgs/nixos-23.11";
}
```

`.`을 사용하는 것은 더 깊은 속성 집합의 중첩된 속성에 대한 바로 가기일 뿐입니다. 따라서 이제 `inputs`의 각 값이 최소한 `url`을 포함하는 새로운 속성 집합 자체임을 인식할 수 있습니다. 이 `url` 속성은 종종 Github 링크이지만 다른 것으로 설정할 수도 있습니다.

이 `.` 바로 가기를 사용하는 경우 여러 번 또는 중첩의 어느 지점에서든 사용할 수 있습니다. 예상대로 모두 결합됩니다.

```nix
{
  # 우리 Flake에 있는 것과 동일합니다!
  home-manager.url = "github:nix-community/home-manager/release-23.11";
  home-manager.inputs.nixpkgs.follows = "nixpkgs";

  # 이것도 동일합니다!
  home-manager = {
    url = "github:nix-community/home-manager/release-23.11";
    inputs = {
      nixpkgs = {
        follows = "nixpkgs";
      };
    };
  };
}
```

`nixpkgs`와 `home-manager`의 실제 이름은 임의적입니다. 원하는 대로 지정할 수 있지만 이러한 이름을 사용하는 것이 일반적이므로 전통을 따르는 것이 가장 좋습니다.

`home-manager` 입력을 보면 `inputs.nixpkgs.follows`라는 속성이 있음을 알 수 있습니다. 이것은 `home-manager` 입력이 `nixpkgs` 입력에 의존한다는 것을 나타내는 방법입니다.

`inputs` 속성에 있을 수 있는 다른 항목에 대한 참조는 [공식 문서](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake.html#flake-inputs)를 참조하십시오. 그러나 일반적으로 초기에는 이것을 너무 많이 건드리지 않는 것이 좋으므로 여기에 있는 두 입력이 우리가 고정하고 있는 Github 리포지토리라는 것을 이해하는 데 편안하다면 계속 진행해도 괜찮습니다.

#### `nixpkgs`

`nixpkgs`는 약간 특별합니다. 80,000개 이상의 패키지를 포함하는 [Nixpkgs 리포지토리](https://github.com/NixOS/nixpkgs)에 대한 참조입니다. 너무 특별해서 Github 정보 없이 자체 가져오기를 수행합니다. `23.11` 부분은 Git 태그입니다. 궁금하다면 [여기서](https://github.com/NixOS/nixpkgs/releases/tag/23.11) 볼 수 있습니다. 그 외에는 특별한 것이 없으며 모든 패키지를 특정 버전으로 고정하는 태그일 뿐입니다. 다른 태그를 사용하거나 특정 Git 커밋 SHA를 제공하거나 최신 버전을 얻고 싶다면 `unstable`로 이동할 수도 있습니다(첫 번째 적용 시 '최신'이었던 것에 여전히 고정된다는 점을 기억하십시오!).

하지만 패키지만 있는 것은 아닙니다! `nixpkgs/lib`에 있는 Nix를 위한 [광범위한 표준 라이브러리](https://nixos.org/manual/nixpkgs/stable/#sec-functions-library)와 같은 것을 포함합니다. 정말 토끼굴을 원한다면 [여기 소스를 확인하십시오](https://github.com/NixOS/nixpkgs/tree/master/lib).

기본적으로 `nixpkgs`는 아마도 여러분이 포함하게 될 가장 중요한 입력일 것이며, 아마도 여러분이 만들 모든 Flake에 포함하게 될 것입니다.

## 지금까지의 요약

자, 이미 많은 내용이 있었고 아직 `inputs`를 지나치지도 못했습니다! 잠시 쉬면서 지금까지 진행된 내용을 이해했는지 확인해 봅시다. `outputs` 섹션은 훨씬 더 복잡해질 것이기 때문입니다.

다음에 대한 일반적인 이해가 있어야 합니다. 그렇지 않다면 돌아가서 다시 읽으십시오.

- Flake란 무엇인가 (기본적으로)
- `inputs`란 무엇이며 어떻게 정의되는가 (기본적으로)
- 몇 가지 기본 Nix 구문
  - Nix 속성 집합은 중괄호 안의 일부 키/값입니다.
  - Nix의 문자열 리터럴
  - 바로 가기로서의 `thing.another.stuff = "value";`

기분이 좋다면 [출력 섹션](./04-explain-outputs-function_ko.md)으로 넘어갑시다.
