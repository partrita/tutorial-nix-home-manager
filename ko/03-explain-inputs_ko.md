# 03 - 설명: 기본 Nix 구문 및 flake.nix 입력

[<- ./02-basic-repository-setup_ko.md](./02-basic-repository-setup_ko.md) | [04 - 설명: flake.nix 출력 함수 ->](./04-explain-outputs-function_ko.md)

첫 번째 Home Manager Flake 설정을 완료하고 저와 함께해 주셔서 감사합니다! 그 노고에 대한 보상으로, 이제 우리가 설정한 모든 것들을 실제로 파헤쳐서 설명해 드릴게요. 작동하는 예시와 함께 맥락을 이해하는 것이 정말 중요하니까요.

제가 지금부터 드릴 설명은 **개괄적인 내용**입니다. Nix의 토끼굴은 여전히 깊으므로 아직 익숙지 않은 상태에서는 얼마나 깊이 파고들지 신중해야 합니다. 이 작은 파일들 안에 얼마나 많은 설명이 필요한지 보면 천천히 진행해야 한다는 사실에 안심이 될 겁니다.

관련 문서나 추가 정보 링크도 포함할 거예요. 살펴보는 것은 좋지만, **길을 잃지 않도록 조심하세요.** 나중에 Nix에 더 익숙해지면 다시 방문할 수 있도록 저장해 두는 것이 좋습니다.

> 이 섹션들을 처음 읽을 때 바로 이해하지 못해도 괜찮습니다. 나중에 더 많이 수정하면서 참고용으로 다시 돌아오세요. 여기서 중요한 점은 이러한 개념들이 나중에 여러분의 학습을 위한 참조점으로 존재한다는 것입니다.


## Flake란 무엇인가요?

"Flake"라는 용어는 Nix의 눈송이 로고에서 따온 것입니다. 그 이상의 심오한 의미는 없어요. 누군가 알려줄 때까지 저도 한동안 궁금해했답니다.

기본적으로 ["Flake"](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake.html)는 Nix에서 **완전히 독립적인 무언가를 정의하는 방법**입니다. 모든 입력은 **`inputs` 속성에 명시적으로 정의**되며, 특정 Git 커밋이나 어떤 종류의 해시에 **고정(pinning)**됩니다. 일반적인 Nix에서는 이 모든 해시를 직접 선언해야 해서 꽤나 번거롭습니다. Flake가 "실험적"이라고 표시되었음에도 불구하고 인기가 많은 이유 중 하나가 바로 이것이죠.

Flake의 **출력(outputs)**에는 여러 가지가 포함될 수 있습니다. 우리가 만든 Flake의 경우, 사용자 공간 환경을 정의하는 데 **Home Manager**가 사용하는 출력을 정의하고 있습니다. 다른 경우에는 패키지, NixOS의 전체 시스템 설정, 또는 단일 함수나 함수 라이브러리 같은 것을 정의할 수도 있습니다. 지금은 우리가 Home Manager 설정을 만들고 있다는 점만 신경 쓰면 됩니다. 나중에는 특정 명령어가 다른 출력 속성을 찾을 수도 있어요.

Flake를 빌드하거나 사용할 때 **`flake.lock` 파일**을 참조하려고 합니다. 이 파일이 없으면, 정의된 입력의 최신 버전을 가져와 잠금 파일을 생성합니다. 입력이 추가되면 `flake.lock` 파일에 자동으로 저장됩니다. 입력이 제거되면 `flake.lock`에서 자동으로 제거되고요. 재현성을 유지하기 위해 자동으로 업데이트되지는 않지만, [Nix 명령으로 명시적으로 업데이트](https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake-update.html)할 수 있습니다.



## Flake의 구조

이제 우리가 만든 Flake를 섹션별로 자세히 살펴봅시다.

### 중괄호 {}

네, 중괄호입니다. 여기서부터 시작해 볼까요.

```nix
{
  # 왜 이게 중괄호 안에 있을까요?
}
```

Nix에서 모든 `.nix` 파일은 단일 **Nix 표현식(Nix expression)**입니다. 이 경우, 우리는 **Nix 속성 집합(Nix attribute set)**을 정의하고 있습니다. Nix 속성 집합은 기본적으로 JSON 객체처럼 키-값 쌍으로 이루어진 사전(dictionary)과 비슷합니다. 여러 속성을 포함하며, 각 속성은 다양한 표현식 값을 가질 수 있습니다.

속성 집합은 중괄호 `{}`로 정의됩니다. 속성 이름은 영숫자 값일 수 있고, 대시, 밑줄 또는 아포스트로피를 포함할 수도 있습니다. 특정 설정을 위해 자체적으로 속성 집합을 가질 때가 많아서 깊게 중첩되는 경우가 흔합니다.

이것들은 Nix에서 정말 흔하게 볼 수 있고, Nix의 핵심이라고 할 수 있습니다.

### Flake의 최상위 속성

Flake에는 우리가 신경 써야 할 몇 가지 최상위 속성이 있습니다. 구체적으로 **`description`**, **`inputs`**, **`outputs`**입니다.

  * `description`은 Flake를 설명하는 **문자열**입니다. 선택 사항입니다.
  * `inputs`는 Flake가 사용하는 **입력 집합**입니다. Flake는 `inputs`에 있는 것과 로컬 Git 저장소에 명시적으로 추가된 것만 인식합니다.
  * `outputs`는 정의된 모든 `inputs`를 받아 우리가 관심 있는 설정을 생성하는 **함수**입니다.

### `description`

```nix
{
  description = "내 Home Manager 구성";
}
```

여기에는 문자열 리터럴을 담고 있는 `description`이라는 속성이 있습니다. 가장 간단한 형태죠. **세미콜론으로 끝나야 한다는 점**에 유의하세요. Nix의 속성 집합에 있는 모든 할당은 세미콜론으로 끝나야 합니다. 이는 공백이 그다지 중요하지 않다는 뜻이기도 합니다.

```nix
{
  # 이것도 괜찮습니다!
  description =
    "내 Home Manager 구성";
}
```

이 `description`은 어디에 사용될까요? `nix flake info`를 실행하면 볼 수 있지만, 솔직히 선택 사항입니다. 단순히 간단한 문자열 값 할당을 보여주기 편리해서 포함시킨 것뿐입니다! 실제로 여러분의 루트 `flake.nix` 파일을 보면 `description` 속성이 전혀 정의되어 있지 않은 것을 알 수 있을 겁니다. 공유 가능한 패키지에서 더 유용하므로, Home Manager 설정에서는 원한다면 생략할 수 있습니다.

### `inputs`

이제 좀 더 흥미로운 부분으로 넘어가 봅시다.

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

이제 `inputs`가 **Nix 속성 집합**으로 설정되어 있다는 것을 알아차렸을 겁니다. `inputs` 안에는 `nixpkgs`와 `home-manager`라는 두 가지 속성이 있습니다.

그런데 잠깐, 이 `nixpkgs.url`이라는 건 뭘까요?

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

온점(`.`)을 사용하는 것은 더 깊은 속성 집합의 중첩된 속성에 대한 **단축 문법**일 뿐입니다. 따라서 이제 `inputs`의 각 값이 최소한 `url`을 포함하는 새로운 속성 집합 자체라는 것을 알 수 있을 겁니다. 이 `url` 속성은 주로 Github 링크이지만, 다른 것으로 설정할 수도 있습니다.

이 온점(.) 단축 문법은 여러 번 또는 중첩된 어떤 지점에서도 사용할 수 있습니다. 예상대로 모두 결합됩니다.

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

`nixpkgs`와 `home-manager`의 실제 이름은 **임의적**입니다. 원하는 대로 지정할 수 있지만, 일반적으로 이 이름을 사용하므로 전통을 따르는 것이 가장 좋습니다.

`home-manager` 입력을 보면 `inputs.nixpkgs.follows`라는 속성이 있는 것을 알 수 있습니다. 이것은 `home-manager` 입력이 `nixpkgs` 입력에 의존한다는 것을 나타내는 방법입니다.

`inputs` 속성에 있을 수 있는 다른 항목에 대한 참조는 [공식 문서](https://www.google.com/search?q=https://nixos.org/manual/nix/stable/command-ref/new-cli/nix3-flake.html%23flake-inputs)를 참조하세요. 하지만 일반적으로 처음에는 이 부분을 너무 많이 건드리지 않는 것이 좋습니다. 그러니 여기에 있는 두 입력이 우리가 고정(pinning)하고 있는 Github 저장소라는 점을 이해했다면 계속 진행해도 괜찮습니다.

#### `nixpkgs`

`nixpkgs`는 약간 특별합니다. 80,000개 이상의 패키지를 포함하는 [Nixpkgs 저장소](https://github.com/NixOS/nixpkgs)에 대한 참조입니다. 너무 특별해서 Github 정보 없이 자체적으로 가져오기를 수행합니다. `23.11` 부분은 Git 태그입니다. 궁금하다면 [여기서](https://github.com/NixOS/nixpkgs/releases/tag/23.11) 볼 수 있습니다. 그 외에는 특별한 것이 없으며, 모든 패키지를 특정 버전으로 고정하는 태그일 뿐입니다. 다른 태그를 사용하거나 특정 Git 커밋 SHA를 제공할 수도 있고, 최신 버전을 얻고 싶다면 `unstable`로 지정할 수도 있습니다 (단, 첫 적용 시 '최신'이었던 버전에 여전히 고정된다는 점을 기억하세요!).

하지만 패키지만 있는 것은 아닙니다! `nixpkgs`는 `nixpkgs/lib`에 있는 Nix를 위한 [광범위한 표준 라이브러리](https://www.google.com/search?q=https://nixos.org/manual/nixpkgs/stable/%23sec-functions-library)와 같은 것도 포함합니다. 정말 토끼굴로 깊이 들어가고 싶다면 [여기 소스 코드를 확인해 보세요](https://github.com/NixOS/nixpkgs/tree/master/lib).

기본적으로 `nixpkgs`는 여러분이 포함하게 될 가장 중요한 입력일 것이며, 아마도 만들게 될 모든 Flake에 포함될 겁니다.



## 지금까지의 요약

자, 벌써 많은 내용을 다뤘고 아직 `inputs` 섹션도 다 지나지 못했습니다! 잠시 쉬면서 지금까지 진행된 내용을 이해했는지 확인해 봅시다. `outputs` 섹션은 훨씬 더 복잡해질 테니까요.

다음 내용에 대해 **대략적으로 이해**하고 있어야 합니다. 그렇지 않다면 돌아가서 다시 읽어보는 게 좋습니다.

  * **Flake란 무엇인가** (기본적인 개념)
  * **`inputs`란 무엇이며 어떻게 정의되는가** (기본적인 개념)
  * **몇 가지 기본적인 Nix 구문**
      * Nix 속성 집합은 중괄호 `{}` 안의 키/값 쌍입니다.
      * Nix의 문자열 리터럴
      * `thing.another.stuff = "value";`와 같은 단축 문법


이제 [출력 섹션](./04-explain-outputs-function_ko.md)으로 넘어갑시다.
