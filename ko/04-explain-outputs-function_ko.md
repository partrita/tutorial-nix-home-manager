# 04 - 설명: Flake.nix 출력 함수 구문

[<- ./03-explain-inputs_ko.md](./03-explain-inputs_ko.md) | [05 - 설명: Flake.nix 출력 함수 본문 ->](./05-explain-outputs-body_ko.md)

지난 섹션에서는 몇 가지 기본적인 Nix 구문과 `description` 및 `inputs` 속성에 대해 살펴보았습니다.

다시 한번 강조하지만, 이제 다음 내용을 이해해야 합니다:

  * **Nix 속성 집합**이란 무엇인가
  * **`.` 표기법**이 작동하는 방식
  * **`inputs` 속성**이 임의의 속성을 가진 Nix 속성 집합이라는 것


[만약 그렇지 않다면, 이전 섹션으로 돌아가세요.](../03-explain-inputs_ko.md#지금까지의-요약)

이제 더 복잡한 **`outputs` 속성**을 다루어 보겠습니다. 더 많은 Nix 구문을 배워야 하므로, 새로운 구문과 이것이 `outputs` 속성에 어떻게 적용되는지 사이를 오가며 설명할 거예요. 내부의 `homeConfigurations` 속성은 다음 섹션으로 미루고, 이번에는 이 몇 줄 안에 많은 일이 일어나고 있으므로 외부 부분에만 집중할 것입니다.

이번 섹션은 길어질 겁니다. 만약 이것이 Nix 문서의 토끼굴에 직접 뛰어들지 않은 것이 현명한 결정이었다는 확신을 주지 못한다면, 그 어떤 것도 그러지 못할 거예요!


## `outputs` 속성

간단히 복습하자면, 현재 `outputs` 전체는 다음과 같습니다:

```nix
{
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
}
```

이제부터 정말 재미있을 거예요. 풀어야 할 내용이 많거든요.


## Nix 함수

Nix가 단순히 보기 좋게 꾸며진 JSON 형식에 불과하다면 지루할 겁니다. 하지만 Nix는 완전한 **함수형 언어**입니다. 그러니 Nix 함수가 어떻게 생겼는지 살펴봅시다. 이 함수는 아직 이름이 지정되거나 어디에서도 사용할 수 없습니다. 실제로 실용적인 함수를 정의하는 방법은 나중에 다룰 거예요. 지금은 구문에만 집중하세요. 어디에나 중첩되어 있는 것을 보게 될 테니까요.

```nix
# 인수
# |
# v
  a: a + 2
#     ^^^^^
#       |
#       결과
```

여기에는 단일 인수 `a`가 있습니다. 결과는 `a + 2`입니다. 간단하죠!

Nix 함수는 항상 **단일 인수**만 가질 수 있습니다. 이를 통해 커링(currying)과 같은 멋진 작업을 많이 수행할 수 있지만, 이건 나중에 다루고, 지금은 함수에 항상 정확히 하나의 인수가 있다는 것만 알아두세요.

함수는 인수로 **속성 집합**을 사용할 수 있으며(종종 사용합니다), Nix 구문을 사용하면 들어오는 속성 집합에서 예상되는 특정 속성으로 분해(destructure)할 수 있습니다. 이렇게 하면 다른 멋진 기본 구문을 사용하지 않는 한 입력 속성을 *반드시* 제공해야 합니다 (이것도 나중에 다룰 내용입니다).

```nix
# 속성 `a`와 `b`를 가진 속성 집합을 가져와 `a + b`를 반환하는 함수입니다.

# 인수 (여전히 단일!)
#    |
# vvvvvvvv
  { a, b }: a + b
#          ^^^^^
#            |
#            결과
```

이 함수의 모든 호출은 속성 집합에 정확히 `a`와 `b`를 제공해야 합니다. 다른 것은 제공할 수 없습니다.

Nix 파일은 항상 단일 표현식만 가질 수 있다는 점을 기억하세요. 즉, 나중에 호출할 전역 범위에서 함수를 정의하지 않을 겁니다. 그렇다면 어디서 오는 걸까요? 어떻게 호출할까요? 계속 진행하면서 보게 될 겁니다. 이미 걱정해야 할 정보가 너무 많아요!

지금 알아야 할 중요한 사항:

  * 함수에는 항상 **정확히 하나의 인수**가 있습니다.
  * 함수는 인수로 **속성 집합**을 사용할 수 있습니다.
  * 함수는 항상 **단일 표현식**으로 평가됩니다.


## `outputs` 함수

자, 드디어 `outputs` 속성으로 돌아가겠습니다. 구체적으로 첫 번째 줄입니다.

```nix
#           인수
#             |
#       vvvvvvvvvvvvvvvvvvvvvvvvvvvv
outputs = { nixpkgs, home-manager, ... }:
```

우리는 `outputs` 속성에 함수를 할당하고 있습니다! 끝에 있는 콜론(`:`)으로 알 수 있죠. 이 함수는 `{ a, b }: a + b` 예시와 유사하게 **속성 집합인 단일 인수**를 사용합니다. 이 속성 집합에는 최소한 `nixpkgs`와 `home-manager` 속성이 있어야 하며, `...`은 "그리고 다른 모든 것도 괜찮지만 우리는 무시할 것입니다"라고 말하는 특별한 Nix 방식입니다.

`nixpkgs`와 `home-manager`는 어디서 왔을까요? 물론 `inputs`입니다! `inputs`의 속성 이름은 임의적이며 `outputs` 함수의 인수로 그대로 전달됩니다. `nixpkgs`를 `idk`로 이름을 바꾸고 다시 빌드해 볼 수 있지만, `outputs` 함수의 인수도 `idk`로 이름을 바꾸기 전까지는 작동하지 않을 겁니다.

이 인수들은 무엇을 담고 있을까요? 마법입니다!

기술적으로는 `inputs`로 전달된 Flake의 `outputs` 결과를 담고 있지만, 그 내용은 다른 날을 위해 미뤄두고 지금은 여기서 일어나는 일을 그냥 믿는 것이 좋습니다.


## `let in` 구문

모든 Nix 표현식은 단일 표현식이어야 하므로, 괄호가 많으면 매우 빠르게 장황해질 수 있습니다. 그래서 대신 **`let in` 구문**을 사용하여 중간 변수를 만들 수 있습니다. 간단히 말해서 `let in` 구문은 다음과 같습니다:

```nix
# 숫자 4로 평가됩니다.
let a = 2; in a + 2
```

`let` 안에 여러 변수를 만들고 공백을 사용할 수 있습니다.

```nix
# 숫자 6으로 평가됩니다.
let
  a = 2;
  b = 3;
in a * b
```

동일한 `let` 블록 내에서 해당 변수를 다시 사용할 수도 있습니다.

```nix
# 숫자 7로 평가됩니다.
let
  a = 2;
  b = a + 1;
in a + b + 2
```

하지만 여기서 선언된 것은 해당 표현식 외부에서 사용할 수 없습니다.

```nix
{
  # 7로 평가됩니다.
  first = let
    a = 2;
    b = a + 3;
  in a+b;

  second = a+3; # 이것은 실패합니다!
}
```

함수도 표현식이라는 점을 기억하세요. 이것이 우리가 함수를 직접 정의하고 사용하는 한 가지 방법입니다.

```nix
# 숫자 8로 평가됩니다.
let
  double = a: a * 2;
in double 4
```

이것은 앞으로 많이 사용하게 될 매우 강력한 도구입니다! 하지만 지금은 이 전체 `let in`이 `outputs` 함수에 대해 무엇을 의미하는지 살펴보겠습니다.


## `outputs`에 대한 `let in`

이제 다음을 이해해야 합니다:

  * `let in`은 다음 표현식 내에서 사용할 수 있는 일부 중간 값을 정의합니다.
  * `let in`에 정의된 것은 동일한 `let in` 블록 내에서 사용할 수 있습니다.

이를 염두에 두고 `outputs` 함수의 `let in` 블록을 살펴보겠습니다:

```nix
let
  lib = nixpkgs.lib;
  system = "x86_64-linux";
  pkgs = import nixpkgs { inherit system; };
in { #...
```

자, 하나씩 살펴보겠습니다.

```nix
lib = nixpkgs.lib;
```

이전에 `nixpkgs`에는 Nix를 위한 거대한 표준 라이브러리 같은 것도 포함되어 있다고 말씀드린 것을 기억하시나요? **[이것이 바로 그 표준 라이브러리입니다.](https://nixos.org/manual/nixpkgs/stable/#id-1.4)** 즐겨찾기에 추가하고 싶을 수도 있지만, 지금은 그냥 한 번 훑어보고 거기에 정말 많은 기능이 있다는 것만 알아두세요. 정말 더 깊이 파고들고 싶다면 [Github 저장소](https://github.com/NixOS/nixpkgs/tree/master/lib)를 확인할 수 있습니다.

이제 `lib`를 정의했으니, 다음을 살펴보겠습니다:

```nix
system = "x86_64-linux";
```

이것은 간단한 문자열 리터럴이므로 익숙할 겁니다. 이 값은 무엇을 의미할까요?

Nix는 [대부분의 Linux 및 Mac 시스템](https://nixos.org/manual/nix/stable/installation/supported-platforms)에서 작동합니다. 하지만 Nix가 여러분의 OS 및 아키텍처에 맞는 올바른 패키지를 가져올 수 있도록 어떤 시스템을 사용하고 있는지 알려야 합니다. 이 경우, 64비트 Linux 시스템을 사용하고 있다고 가정합니다. 그렇지 않다면 직접 찾아본 후 적절한 값으로 변경해야 합니다.

하지만 이 `system` 변수는 마법이 아닙니다. 이 `let` 블록에는 어떤 마법적인 이름도 없으며 모두 임의적입니다. 원한다면 `steve = "x86_64-linux;"`로 설정할 수도 있었죠 (물론 나중에 약간 변경해야 했겠지만요). 그렇다면 어떻게 사용될까요? 다음 줄이 그 답입니다.

```nix
pkgs = import nixpkgs { inherit system; };
```

여기에는 **정말 많은 일**이 일어나고 있으므로, 한 단계씩 살펴보겠습니다.

### `import`란 무엇인가요?

일반적으로 `import`는 실제로 [파일에 사용됩니다](https://nixos.org/manual/nix/stable/language/builtins.html#builtins-import). 나중에 설정을 여러 로컬 파일로 분할하고 싶을 때 실제로 많이 사용하게 될 거예요. 여기에는 약간의 마법이 일어나고 있는데, [이것이 Flake에서 사용할 수 있는 이유를 설명합니다.](https://stackoverflow.com/questions/74464687/whats-the-mechanism-behind-import-nixpkgs-in-nix-flakes)

간단히 말해, 웃으며 고개를 끄덕이고 **`import nixpkgs`가 우리가 호출할 수 있는 함수로 평가된다는 것**을 이해하면 됩니다.

구체적으로 `import nixpkgs`는 이 줄의 나머지 부분보다 먼저 실행됩니다. `let in`에 대한 지식을 고려하면 이 부분이 이해될 겁니다.

```nix
let
  # 이미 하고 있는 것과 동일하며, 추가 단계만 있습니다.
  makePkgs = import nixpkgs;
  pkgs = makePkgs { inherit system; };
in #...
```

#### `import` vs. `legacyPackages` 사용

여담으로, `import` 대신 `legacyPackages`의 일부 속성을 사용하는 예제를 볼 수 있습니다. 둘 다 동일하지만, `legacyPackages`는 여러 번 `import`하는 더 복잡한 설정에서 실제로 더 빠를 수 있습니다. 더 깊이 파고들고 싶다면 [이 스레드를 확인해 보세요.](https://discourse.nixos.org/t/using-nixpkgs-legacypackages-system-vs-import/17462/7) 지금은 `import`를 사용하고 있습니다. 처음부터 `import`를 소개할 수 있었고, 이름에 `legacy`가 있는 것을 사용하는 것에 항상 조심스러웠기 때문입니다. [실제로 더 이상 사용되지 않는 것은 아니고, 단지 명명 문제일 뿐입니다.](https://github.com/NixOS/nixpkgs/blob/master/flake.nix#L64)

### 함수 호출과 `inherit`

`import nixpkgs`가 우리가 호출할 수 있는 함수를 반환한다는 것을 기억하세요. 그리고 위에서 함수 호출은 단일 인수를 사용한다고 했죠. 이 경우 인수는 속성 집합이며 `system`이 필요합니다.

이것을 어떻게 알 수 있을까요? 어디선가 읽고, 우리 모두처럼 복사/붙여넣기하는 겁니다. *Nix에 오신 것을 환영합니다!* 저는 아직 공식 문서에서 해당 정보를 찾을 수 없습니다. 어디에 있는지 안다면 여기에 이슈를 제기해 주세요.

자, 우리는 속성 집합을 제공하고 싶고 `system` 속성을 제공하고 싶습니다. 그렇다면 이 `inherit system`은 무엇을 의미할까요? 기본적으로 `inherit a;`는 `a = a;`와 **정확히 동일**합니다.

```nix
# 이 전체 표현식은 `true`로 평가됩니다.
let
  a = 3;
  # 다음 선언은 정확히 동일합니다 (b == c).
  b = { inherit a; };
  c = { a = a; };
in
  b.a == c.a
```

따라서 우리 예제에서 `inherit system`은 단지 `system = system;`이라고 말하는 것입니다. Nix에서 일반적으로 사용되는 단축 문법이며, 많이 보게 될 겁니다. 왜 귀찮게 할까요? 상속할 속성이 여러 개 있을 때 동일한 `inherit` 안에 계속 나열할 수 있기 때문에 더 유용합니다. 다음과 같이요:

```nix
let
  someReallyImportantNumber = 3;
  anotherReallyImportantNumber = 4;
  # 다음 선언은 정확히 동일합니다 (a == b).
  a = { inherit someReallyImportantNumber anotherReallyImportantNumber; };
  b = {
    someReallyImportantNumber = someReallyImportantNumber;
    anotherReallyImportantNumber = anotherReallyImportantNumber;
  };
in
  # ...
```

`inherit`을 사용하여 중첩된 속성을 가져오는 멋진 트릭을 수행할 수도 있지만, 그건 다른 날을 위한 겁니다. 지금은 `inherit`의 기본 사항에 익숙해지고, Nix 전문가들처럼 보이고 싶다면 속성 집합에서 `x = x`를 사용할 때마다 `inherit x`를 사용하는 습관을 들이세요.

만약 `system = "x86_64-linux";` 줄을 `steve = "x86_64-linux";`로 이름을 바꾸셨다면, `inherit system`을 `system = steve;`로 변경해야 합니다. `inherit`을 사용하고 더 명확하게 만들기 위해 `let` 블록에서 `system`이라는 이름을 구체적으로 선택했습니다.


## 지금까지의 요약

자, 정말 많은 내용을 다뤘습니다! 모든 것을 종합해 봅시다. 우리는 아직 함수의 외부 구문만 보고 있으며 내부 정의는 보지 않고 있습니다.

```nix
# 다음은 이제 익숙할 겁니다.
outputs = { nixpkgs, home-manager, ... }:
  let
    lib = nixpkgs.lib;
    system = "x86_64-linux";
    pkgs = import nixpkgs { inherit system; };
  in {
    # 내용물
  }
```

다음 내용은 이해되어야 합니다:

  * **`outputs` 속성**은 속성 집합을 인수로 사용하는 Nix 함수입니다.
      * 속성 집합은 `inputs`인 `nixpkgs`와 `home-manager`로 구성됩니다.
      * `...`은 우리가 신경 쓰지 않는 다른 인수가 주어질 수 있음을 의미합니다.
  * `let`을 사용하면 다음 표현식에서 사용할 중간 값을 선언할 수 있습니다.
  * `lib`를 `nixpkgs.lib`에 있는 `nixpkgs` 표준 라이브러리로 선언합니다.
  * 우리 시스템 유형은 `x86_64-linux`입니다.
  * `pkgs`를 `x86_64-linux` 시스템의 모든 패키지를 포함하는 값으로 선언합니다.
  * **`in`** 다음의 표현식에서 `lib`, `system`, `pkgs` 중 어느 것이든 사용할 수 있습니다 (`# 내용물`에 있는 모든 것).

위 내용 중 이해가 되지 않는 부분이 있다면 해당 섹션을 다시 읽거나, 더 궁금한 점이 있다면 언제든지 질문해 주세요.

[다음](./05-explain-outputs-body_ko.md)에서는 마침내 실제 `homeConfigurations` 속성과 내부 `home-manager` 함수를 다룰 것입니다.
