# 05 - 설명: Flake.nix 출력 함수 본문

[<- 설명: Flake.nix 출력 함수 구문](04-explain-outputs-function_ko.md)| [설명: Home.nix ->](06-explain-home-nix_ko.md)

지난 섹션에서는 `flake.nix`의 **`outputs`** 속성을 처음 살펴보았습니다. 이제 다음을 이해해야 합니다:

  * **Nix 함수 정의 구문**
  * **`let in` 구문**이란 무엇인가
  * `inherit`이 의미하는 것


이 섹션에서는 **`outputs` 함수가 실제로 수행하는 작업**을 살펴봄으로써 초기 `flake.nix`에 대한 설명을 마무리할 것입니다.

참고로 전체 내용은 다음과 같습니다:

```nix
outputs = { nixpkgs, home-manager, ... }:
  let
    lib = nixpkgs.lib;
    system = "x86_64-linux";
    pkgs = import nixpkgs { inherit system; };
  in {

    homeConfigurations = {
      myprofile = home-manager.lib.homeManagerConfiguration {
        inherit pkgs;
        modules = [ ./home.nix ];
      };
    };
  };
```


## `outputs`는 무엇을 생성해야 하나요?

먼저, 또 다른 퀴즈입니다\! 현재 지식으로는 이 표현식을 이해할 수 있어야 합니다:

```nix
{ a, b }: { sum = a + b; };
```

이 함수는 무엇을 할까요? 먼저 생각해 본 다음 설명을 읽어보세요.


이 함수는 **단일 인수**를 받습니다. 그 인수는 `a`와 `b` 속성을 포함하는 **속성 집합**입니다. 함수는 `a + b`와 같은 단일 속성 `sum`을 포함하는 **새로운 속성 집합**을 반환합니다.

이해하셨나요? 좋습니다. 이제 출력 함수를 다시 살펴보되, 실제로 구성하는 내용에 집중하기 위해 불필요한 부분은 제거해 보겠습니다.

```nix
# 대폭 단순화됨
outputs = { nixpkgs, home-manager, ... }:
  {
    homeConfigurations = "<무언가>";
  };
```

그렇다면 이것은 무엇을 의미할까요? `flake.nix` 파일에 의해 정의된 속성 집합의 **`outputs` 속성**은 **함수**라는 것을 의미합니다. 해당 함수는 `nixpkgs`와 `home-manager`를 포함하는 속성 집합을 사용하며, 우리가 신경 쓰지 않는 다른 임의의 속성도 허용합니다. 이 함수는 단일 속성 `homeConfigurations`를 포함하는 속성 집합을 생성합니다.

이것은 위에서 살펴본 `{ a, b }: { sum = a + b; }` 함수와 동일한 구조입니다. 구문 관점에서 특별할 것은 없습니다.

여기서 중요한 점은 **`outputs`는 `inputs`로 구성된 속성 집합을 사용하는 함수여야 하며, 새 속성 집합을 생성해야 한다**는 것입니다.

해당 출력 속성 집합의 값은 무엇이어야 할까요? Flake가 사용되는 용도에 따라 다릅니다. Nix는 실제로 살펴볼 가치가 있는 [다양한 특수 속성](https://nixos.wiki/wiki/Flakes#Output_schema)에 대해 알고 있습니다. 특히 `nixosConfigurations`는 NixOS 시스템을 정의하려는 경우 원하는 것입니다. 하지만 오늘은 그중 어느 것도 원하지 않으며, 해당 목록 끝에 다음과 같이 표시되어 있음을 알 수 있습니다:

> 추가적인 임의의 속성을 정의할 수도 있지만, 이것들은 Nix가 알고 있는 출력입니다.

우리의 특정 경우에는 Home Manager가 사용할 수 있는 정의를 만드는 데만 관심이 있습니다. 이것은 출력 속성 집합에서 **`homeConfigurations` 속성을 정의**하여 수행됩니다.

`homeConfigurations`라는 것을 어떻게 알 수 있을까요? 다른 곳에서 복사/붙여넣기하는 겁니다. [공식 문서](https://nix-community.github.io/home-manager/index.xhtml#sec-flakes-standalone)에서는 `home-manager init`을 실행하라고 하는데, 결국에는 그 내용에 도달하게 될 것이지만, 솔직히 이 글을 쓰는 시점(2024년 초)에는 이 정보가 어딘가에 쉽게 정리되어 있지 않다는 것이 약간 답답합니다. 다시 말하지만, Nix에 오신 것을 환영합니다\!

어쨌든, 이제 복사/붙여넣기할 소스가 있습니다. 그리고 이제 다음을 이해해야 합니다:

  * **`flake.nix`의 `outputs` 속성**은 속성 집합을 생성하는 함수입니다.
  * 해당 속성 집합은 **Flake의 의도된 용도**에 따라 다른 것을 포함할 수 있습니다.
  * 우리의 용도에는 **`homeConfigurations` 속성을 정의**해야 합니다.


## `homeConfigurations` 속성

이제 `flake.nix` 퍼즐의 마지막 조각을 살펴보겠습니다:

```nix
homeConfigurations = {
  myprofile = home-manager.lib.homeManagerConfiguration {
    inherit pkgs;
    modules = [ ./home.nix ];
  };
};
```

### `home-manager switch` 명령 설명

이 경우 `homeConfigurations`는 또 다른 **속성 집합**입니다. `homeConfigurations`의 각 속성은 **사용자 프로필의 이름**입니다. Makefile에서 이 명령을 사용한 것을 눈치채셨을 겁니다:

```bash
home-manager switch --flake .#myprofile
```

이것은 Flake 경로 다음에 `#selector`가 오는 Flake에서 자주 사용되는 구문입니다. 우리의 경우 **`.`은 현재 경로**를 의미합니다. `#myprofile`은 `homeConfigurations` 목록에서 `myprofile`이라는 프로필을 선택하는 것을 의미합니다. 여러 프로필을 갖고 싶다면 `homeConfigurations` 속성 집합에 추가하고 동일한 방식으로 `#`으로 선택할 수 있습니다.

### `myprofile`의 값

`myprofile`의 값은 **함수 호출 결과**로 설정됩니다. 이제 이것을 인식할 수 있어야 합니다:

```nix
#             함수
#       vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
myprofile = home-manager.lib.homeManagerConfiguration { ... }
#                                                     ^^^^^^^
#                                                    인수로서의 단일 속성 집합
```

`nixpkgs`와 마찬가지로 `home-manager`에는 자체 `lib`가 있으며, 이것은 (저와 함께 말하세요\!) 또 다른 **속성 집합**입니다. 값 중 하나인 `homeManagerConfiguration`은 `home-manager`가 사용하는 실제 출력을 생성하는 함수입니다.

이제 이 함수의 입력을 살펴보겠습니다:

```nix
# home-manager.lib.homeManagerConfiguration에 전달되는 인수
{
  inherit pkgs;
  modules = [ ./home.nix ];
}
```

이 `homeManagerConfiguration` 함수의 입력으로, 설치할 수 있는 모든 잠재적 패키지의 정의와 우리가 직접 정의한 실제 설정을 제공해야 합니다. 일반적으로 이것은 이전에 우리 시스템의 모든 `nixpkgs`에 대해 Home Manager에 알리기 위해 정의한 `pkgs`가 될 겁니다. 다시 한번 우리는 중간 변수의 이름을 편리하게 `pkgs`로 지정했으므로 `inherit`을 사용합니다. 다시 말하지만, 이것은 Nix 규칙을 따르기 위해 `pkgs = pkgs;`라고 말하는 다른 방법일 뿐입니다.

또한 **`modules`** 목록을 제공해야 합니다. 여기서는 `home.nix` 파일을 제공합니다. '모듈'이 무엇인지는 나중에 이야기하겠지만, 지금은 `home.nix`가 실제로 '모듈'이며 이러한 '모듈'이 설치할 패키지, 관리할 파일 등과 같은 **설정 정보를 제공한다**는 것만 알아두세요.

몇 가지 추가 패키지를 포함하도록 `home.nix`를 수정하려고 했을 때 **리스트 구문**을 이미 경험하셨을 겁니다. 구문을 간단히 복습하자면 다음과 같습니다:

```nix
{
  # 문자열의 nix 목록
  mylist = [ "a" "b" "c" ];
}
```

이 경우 전달되는 모듈은 하나뿐입니다.

### 경로

`./home.nix`가 문자열로 제공될 것이라고 예상했을 수도 있습니다. 그러나 Nix는 패키징, 설정 등을 처리하기 위해 특별히 제작된 언어입니다. 그렇기 때문에 경로는 실제로 [일급 데이터 유형](https://nixos.org/manual/nix/stable/language/values#type-path)입니다.

일반적으로 사용하는 모든 경로는 Flake 루트에 대한 **상대 경로**입니다. 실제 파일 이름 `home.nix`는 중요하지 않습니다. `modules/home.nix` 또는 `configurations/myprofile/home.nix`와 같이 하위 디렉터리에 넣을 수도 있습니다. 점점 더 많은 설정 코드를 관리하게 되면 이렇게 정리하기 시작할 겁니다.

#### 여담: 마법의 파일 이름

알아두면 좋은 **두 가지 마법의 파일 이름**이 있습니다. 첫 번째는 `flake.nix`입니다. 예상대로 다양한 Flake 명령에서 사용하는 파일입니다.

두 번째는 `default.nix`입니다. 디렉터리를 가져오면 Nix는 해당 디렉터리에서 `default.nix` 파일을 찾습니다. 이렇게 하면 가져오기가 깔끔해지고 나중에 탐색할 가치가 있을 수 있습니다.

하지만 지금은 `home.nix`로 충분합니다.


## 요약

이제 전체 `flake.nix` 파일을 살펴보았습니다. 이 섹션에서 다음을 이해해야 합니다:

  * **`outputs` 함수**는 속성 집합을 생성합니다.
      * 해당 속성 집합은 **Flake 사용 용도**에 따라 다른 것을 포함할 수 있습니다.
  * **Home Manager**는 **`homeConfigurations` 속성**을 찾습니다.
      * 각 속성은 **단일 사용자 프로필**을 나타냅니다.
      * 이 속성은 `home-manager.lib.homeManagerConfiguration`을 사용하여 생성됩니다.
  * `home-manager switch --flake .#프로필이름`을 사용하여 사용하려는 프로필을 선택할 수 있습니다.

이 시점에서 `flake.nix`의 각 줄을 보고 **어느 정도 무엇을 의미하는지 익숙해져야 합니다.** 처음부터 이것을 작성하거나 각 줄에 대한 심층적인 설명을 할 수 있을 필요는 없습니다. 한 줄을 보고 "네, 저것이 X라는 것을 알겠습니다."라고 생각할 수 있다면 충분하며, 더 많이 가지고 놀면서 나중에 언제든지 다시 돌아올 수 있습니다.

대부분의 경우 **`flake.nix`를 자주 편집하지는 않을 겁니다.** 나중에 새 프로필을 추가하거나 일부 입력을 추가하는 경우에만 다시 이 파일을 보게 될 거예요.


이로써 `flake.nix`에 대한 설명이 마무리되었습니다. 다음으로 [home.nix](./06-explain-home-nix_ko.md) 파일에 대해 알아보겠습니다.
