# 04 - 설명: Flake.nix 출력 함수 구문

[<- 03 - 설명: 기본 Nix 구문 및 flake.nix 입력](./03-explain-inputs_ko.md) | [05 - 설명: Flake.nix 출력 함수 본문 ->](./05-explain-outputs-body_ko.md)

지난 섹션에서는 몇 가지 기본 Nix 구문, `description` 속성 및 `inputs` 속성에 대해 다루었습니다.

다시 말하지만, 이제 다음을 이해해야 합니다.

- Nix 속성 집합이란 무엇인가
- `.` 표기법이 작동하는 방식
- `inputs` 속성이 임의의 속성을 가진 Nix 속성 집합이라는 것

[그렇지 않다면 돌아가십시오.](./03-checkpoint-explanation_ko.md)

이제 더 무서운 `outputs` 속성을 다루어 보겠습니다. 더 많은 Nix 구문을 배워야 하므로 새 구문과 이것이 `outputs` 속성에 어떻게 적용되는지 사이를 오가게 될 것입니다. 내부 `homeConfigurations` 속성은 다음 섹션으로 남겨두고, 이 몇 줄에 많은 일이 일어나고 있으므로 외부 비트에만 집중할 것입니다.

길어질 것입니다. 이것이 Nix 문서의 토끼굴에 스스로 뛰어들지 않은 것이 옳았다는 것을 확신시켜주지 못한다면 아무것도 없을 것입니다.

## 출력

간단히 복습하자면, 현재 `outputs` 전체는 다음과 같습니다.

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

재미있을 준비가 되었기를 바랍니다. 여기서 풀어나갈 것이 많기 때문입니다.

## Nix 함수

지금까지 본 것처럼 Nix가 단순히 미화된 JSON 형식이라면 지루할 것입니다. 하지만 Nix는 완전한 함수형 언어이므로 Nix의 함수가 어떻게 생겼는지 살펴보겠습니다. 이것은 아직 이름이 지정되거나 어디에서도 사용할 수 없습니다. 실제로 실용적인 함수를 정의하는 방법은 나중에 다룰 것입니다. 지금은 구문에만 집중하십시오. 모든 곳에서 중첩되어 있는 것을 볼 수 있기 때문입니다.

```nix
# 인수
# |
# v
  a: a + 2
#    ^^^^^
#     |
#     결과
```

단일 인수 `a`가 있습니다. 결과는 `a + 2`입니다. 간단합니다!

Nix 함수에는 항상 단일 인수만 있을 수 있습니다. 이를 통해 커링과 같은 정말 멋진 작업을 많이 수행할 수 있습니다. 이것은 다른 날을 위해 남겨두고, 지금은 함수에 항상 정확히 하나의 인수가 있다는 것만 알아두십시오.

함수는 인수로 속성 집합을 사용할 수 있으며(종종 사용함) Nix 구문을 사용하면 들어오는 속성 집합을 예상하는 특정 속성으로 _분해_할 수 있습니다. 이렇게 하면 다른 멋진 기본 구문을 사용하지 않는 한 입력 속성을 _반드시_ 제공해야 합니다(이것은 다른 날을 위한 것입니다).

```nix
# 속성 `a`와 `b`를 가진 속성 집합을 가져와 `a + b`를 반환하는 함수입니다.

# 인수 (여전히 단일!)
#    |
# vvvvvvvv
  { a, b }: a + b
#           ^^^^^
#             |
#           결과
```

이 함수의 모든 호출은 속성 집합에 정확히 `a`와 `b`를 제공해야 합니다. 다른 것은 제공할 수 없습니다.

Nix 파일은 항상 단일 표현식일 수만 있다는 것을 기억하십시오. 즉, 나중에 호출할 전역 범위에서 함수를 정의하지 않을 것입니다. 그렇다면 어디서 오는 걸까요? 어떻게 호출할까요? 진행하면서 보게 될 것입니다. 이것은 이미 걱정해야 할 정보가 너무 많습니다!

지금 알아야 할 중요한 사항:

- 함수에는 항상 정확히 하나의 인수가 있습니다.
- 함수는 인수로 속성 집합을 사용할 수 있습니다.
- 함수는 항상 단일 표현식으로 평가됩니다.

## `outputs` 함수

자, 드디어 `outputs` 속성으로 돌아가겠습니다. 구체적으로 첫 번째 줄입니다.

```nix
#           인수
#              |
#         vvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
outputs = { nixpkgs, home-manager, ... }:
```

`outputs` 속성을 함수에 할당하고 있습니다! 끝에 있는 `:`로 알 수 있습니다. 이 함수는 `{ a, b }: a + b` 예제와 유사하게 속성 집합인 단일 인수를 사용합니다. 속성 집합에는 최소한 `nixpkgs`와 `home-manager` 속성이 있어야 하며, `...`은 "그리고 다른 모든 것도 괜찮지만 무시할 것입니다"라고 말하는 특별한 Nix 방식입니다.

`nixpkgs`와 `home-manager`는 어디서 왔을까요? 물론 `inputs`입니다! `inputs`의 속성 이름은 임의적이며 `outputs` 함수의 인수에 그대로 전달됩니다. `nixpkgs`를 `idk`로 이름을 바꾸고 다시 빌드해 볼 수 있지만 `outputs` 함수의 인수도 `idk`로 이름을 바꾸기 전까지는 작동하지 않습니다.

이러한 인수는 무엇을 담고 있을까요? 마법입니다.

기술적으로는 `inputs`로 전달된 Flake의 `outputs` 결과를 담고 있지만, 그 탐색은 다른 날을 위해 남겨두고 지금은 여기서 일어나는 일을 그냥 믿는 것이 좋습니다.

## Let/in 구문

모든 nix 표현식은 단일 표현식이어야 하므로 괄호가 많으면 매우 빠르게 장황해질 수 있습니다. 그래서 대신 `let in`을 사용하여 중간 변수를 만들 수 있습니다. 간단히 말해서 `let in` 구문은 다음과 같습니다.

```nix
# 숫자 4로 평가됩니다.
let a = 2; in a + 2
```

`let`에 여러 변수를 만들고 공백을 사용할 수 있습니다.

```nix
# 숫자 6으로 평가됩니다.
let
  a = 2;
  b = 3;
in a * b
```

동일한 `let`에서 해당 변수를 다시 사용할 수도 있습니다.

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

  second = a+3; # 이것은 실패합니다.
}
```

함수도 표현식이라는 것을 기억하십시오. 이것이 우리가 스스로 함수를 정의하고 사용하는 한 가지 방법입니다.

```nix
# 숫자 8로 평가됩니다.
let
  double = a: a * 2;
in double 4
```

이것은 앞으로 많이 사용할 수 있는 매우 강력한 도구입니다! 하지만 지금은 이 전체 `let in`이 `outputs` 함수에 대해 무엇을 의미하는지 살펴보겠습니다.

## `outputs`에 대한 Let/in

이제 다음을 이해해야 합니다.

- `let in`은 다음 표현식 내에서 사용할 수 있는 일부 중간 값을 정의합니다.
- `let in`에 정의된 것은 동일한 `let in` 블록 내에서 사용할 수 있습니다.

이를 염두에 두고 `outputs` 함수의 `let in` 블록을 살펴보겠습니다.

```nix
let
  lib = nixpkgs.lib;
  system = "x86_64-linux";
  pkgs = import nixpkgs { inherit system; };
in { #...
```

자. 하나씩 살펴보겠습니다.

```nix
lib = nixpkgs.lib;
```

이전에 `nixpkgs`에는 Nix를 위한 거대한 표준 라이브러리 같은 것도 포함되어 있다고 말씀드린 것을 기억하십니까? [이것이 그 표준 라이브러리입니다.](https://nixos.org/manual/nixpkgs/stable/#id-1.4) 즐겨찾기에 추가하고 싶을 수도 있지만 지금은 그냥 한 번 훑어보고 거기에 많은 것이 있다는 것을 확신하십시오. 정말 더 깊이 파고들고 싶다면 [Github 리포지토리](https://github.com/NixOS/nixpkgs/tree/master/lib)를 볼 수 있습니다.

자, `lib`가 있습니다. 다음을 살펴보겠습니다.

```nix
system = "x86_64-linux";
```

이것은 간단한 문자열 리터럴이므로 익숙할 것입니다. 이 값은 무엇에 관한 것일까요?

Nix는 [대부분의 Linux 및 Mac 시스템](https://nixos.org/manual/nix/stable/installation/supported-platforms)에서 작동합니다. 그러나 OS 및 아키텍처에 맞는 올바른 패키지를 가져올 수 있도록 Nix에 어떤 시스템을 사용하고 있는지 알려야 합니다. 이 경우 64비트 Linux 시스템을 사용하고 있다고 가정합니다. 그렇지 않은 경우 직접 찾은 후 적절한 값으로 변경해야 합니다.

하지만 이 `system` 변수는 마법이 아닙니다. 이 `let`에는 어떤 종류의 마법적인 이름도 없으며 모두 임의적입니다. 원한다면 `steve = "x86_64-linux;"`로 설정할 수도 있었습니다(나중에 약간 변경). 그렇다면 어떻게 사용할까요? 다음 줄이 그 답입니다.

```nix
pkgs = import nixpkgs { inherit system; };
```

여기에는 _많은_ 일이 일어나고 있으므로 한 단계씩 살펴보겠습니다.

### `import`란 무엇입니까?

일반적으로 `import`는 실제로 [파일에 사용됩니다](https://nixos.org/manual/nix/stable/language/builtins.html#builtins-import). 나중에 구성을 로컬에서 여러 파일로 분할하고 싶을 때 실제로 많이 사용할 것입니다. 여기에는 약간의 마법이 일어나고 있습니다. [이것은 Flake에서 사용할 수 있는 이유를 설명합니다.](https://stackoverflow.com/questions/74464687/whats-the-mechanism-behind-import-nixpkgs-in-nix-flakes)

간단히 말해서 미소를 짓고 고개를 끄덕이며 `import nixpkgs`가 우리가 호출할 수 있는 함수로 평가된다는 것을 이해하십시오.

구체적으로 `import nixpkgs`는 줄의 나머지 부분보다 먼저 발생합니다. `let in`에 대한 지식을 고려하면 이것이 이해가 될 것입니다.

```nix
let
  # 이미 하고 있는 것과 동일하며 추가 단계만 있습니다.
  makePkgs = import nixpkgs;
  pkgs = makePkgs { inherit system; };
in #...
```

#### `import` 대 `legacyPackages` 사용

여담으로, `import` 대신 `legacyPackages`의 일부 속성을 사용하는 예제를 볼 수 있습니다. 둘 다 동일하지만 `legacyPackages`는 여러 번 `import`하는 더 복잡한 설정에서 실제로 더 빠를 수 있습니다. 괴짜가 되고 싶다면 [이 스레드를 확인하십시오](https://discourse.nixos.org/t/using-nixpkgs-legacypackages-system-vs-import/17462/7). 지금은 `import`를 사용하고 있습니다. 처음부터 `import`를 소개할 수 있었고, 이름에 `legacy`가 있는 것을 사용하는 것에 항상 조심스러웠기 때문입니다. [실제로 더 이상 사용되지 않는 것이 아니라 단지 명명 문제일 뿐입니다.](https://github.com/NixOS/nixpkgs/blob/master/flake.nix#L64)

### 함수 호출 및 `inherit`

`import nixpkgs`가 우리가 호출할 수 있는 함수를 반환한다는 것을 기억하십시오. 그리고 위에서 함수 호출은 단일 인수를 사용한다는 것을 기억하십시오. 이 경우 인수는 속성 집합이며 `system`이 필요합니다.

이것을 어떻게 알 수 있을까요? 어딘가에서 읽고 우리 모두처럼 복사/붙여넣기합니다. _Nix에 오신 것을 환영합니다!_ 저는 여전히 공식 문서에서 해당 정보를 찾을 수 없습니다. 어디에 있는지 안다면 여기에 문제를 제기하십시오.

자, 속성 집합을 제공하고 싶고 `system` 속성을 제공하고 싶습니다. 그렇다면 이 `inherit system`은 무엇에 관한 것일까요? 기본적으로 `inherit a;`는 `a = a;`와 정확히 동일합니다.

```nix
# 이 전체 표현식은 `true`로 평가됩니다.
let
  a = 3;
  # 다음 선언은 정확히 동일합니다 (b == c).
  b = { inherit a; };
  c = { a = a };
in
  b.a == c.a
```

따라서 우리 예제에서 `inherit system`은 단지 `system = system;`이라고 말하는 것입니다. Nix에서 일반적으로 사용되는 바로 가기이며 많이 보게 될 것입니다. 왜 귀찮게 할까요? 상속할 속성이 여러 개 있을 때 동일한 `inherit`에 계속 나열할 수 있기 때문에 더 유용해집니다. 다음과 같이 하십시오.

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

`inherit`을 사용하여 중첩된 속성을 가져오는 멋진 트릭을 수행할 수도 있지만 그것은 다른 날을 위한 것입니다. 지금은 `inherit`의 기본 사항에 익숙해지고 Nix 멋쟁이들과 어울리고 싶다면 속성 집합에서 `x = x`를 사용할 때마다 사용하는 습관을 들이십시오.

실제로 `system = "x86_64-linux";` 줄을 `steve = "x86_64-linux";`로 이름을 바꿨다면 `inherit system`을 `system = steve`로 변경해야 합니다. `inherit`을 사용하고 더 명확하게 만들기 위해 `let` 블록에서 `system`이라는 이름을 구체적으로 선택했습니다.

## 지금까지의 요약

자, 정말 많았습니다! 모든 것을 종합해 봅시다. 우리는 여전히 함수의 외부 구문만 보고 있으며 내부 정의는 보지 않고 있습니다.

```nix
# 다음은 이제 익숙할 것입니다.
outputs = { nixpkgs, home-manager, ... }:
  let
    lib = nixpkgs.lib;
    system = "x86_64-linux";
    pkgs = import nixpkgs { inherit system; };
  in {
    # 내용물
  }
```

다음은 이해가 되어야 합니다.

- `outputs` 속성은 속성 집합을 사용하는 Nix 함수입니다.
  - 속성 집합은 `inputs`인 `nixpkgs`와 `home-manager`로 구성됩니다.
  - `...`은 우리가 신경 쓰지 않는 다른 것이 주어질 수 있음을 의미합니다.
- `let`을 사용하면 다음 표현식에서 사용할 중간 값을 선언할 수 있습니다.
- `lib`를 `nixpkgs.lib`에 있는 `nixpkgs` 표준 라이브러리로 선언합니다.
- 우리 시스템 유형은 `x86_64-linux`입니다.
- `pkgs`를 `x86_64-linux` 시스템의 모든 패키지를 포함하는 값으로 선언합니다.
- `in` 다음의 표현식에서 `lib`, `system`, `pkgs` 중 어느 것이든 사용할 수 있습니다(`# 내용물`에 있는 모든 것).

위의 내용 중 이해가 되지 않는 부분이 있다면 해당 섹션을 다시 읽거나 문제에서 질문하여 명확히 할 수 있습니다.

[다음](./05-explain-outputs-body_ko.md)에서는 마침내 실제 `homeConfigurations` 속성과 내부 `home-manager` 함수를 다룰 것입니다.
