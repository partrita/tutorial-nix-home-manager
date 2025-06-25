# 06 - 설명: Home.nix

[<- 설명: Flake.nix 출력 함수 본문](05-explain-outputs-body_ko.md)

지난 섹션에서는 `flake.nix` 파일에 대한 설명을 완료했습니다. 이제 다음을 이해해야 합니다.

```nix
# 참고용
myprofile = home-manager.lib.homeManagerConfiguration {
  inherit pkgs;
  modules = [ ./home.nix ];
}
```

- 프로필은 `home-manager.lib.homeManagerConfiguration` 함수 호출의 결과입니다.
- 이 함수에 단일 인수를 전달했습니다.
  - `pkgs` 속성은 `nixpkgs`에서 가져온 것이며 모든 64비트 Linux 패키지를 포함합니다.
  - `modules` 속성은 (이 경우) 단일 값을 포함하는 목록이며, 해당 값은 경로입니다.
    - 해당 경로는 `home.nix` 파일입니다.

여기서 여러 "모듈"을 제공할 수 있지만 지금은 단일 진입점만 있습니다.

## 모듈이란 무엇입니까?

그렇다면 모듈이란 무엇일까요? 기본적으로 일부 입력을 받아 일부 출력을 생성하는 독립적인 단일 함수입니다. 구체적으로 `pkgs`와 같은 것을 포함하는 입력 속성 집합을 사용한 다음 설치할 패키지 목록이나 생성할 구성 파일과 같은 것을 포함하는 출력 속성 집합을 생성하며, 이는 NixOS 또는 Home Manager와 같은 일부 도구에서 처리되어 유용한 작업을 수행합니다.

## home.nix의 구조

이 시점에서 많은 단어가 나왔으므로 모듈 자체인 `home.nix` 파일을 살펴보겠습니다.

```nix
{ lib, pkgs, ... }: {
  home = {
    packages = with pkgs; [
      hello
      cowsay
    ];

    username = "myusername";
    homeDirectory = "/home/myusername";

    stateVersion = "23.11";
  };
}
```

`with`를 제외하고 이 구문의 대부분은 익숙할 것입니다. 나중에 다시 살펴보겠습니다.

먼저 몇 가지 전체 구조를 확인해 보겠습니다. 모든 Nix 파일은 [첫 번째 설명 섹션에서 설명한](./03-explain-inputs_ko.md) 단일 표현식이어야 한다는 것을 기억하십니까? 이 경우 이 파일의 단일 표현식은 함수입니다. 이 함수는 단일 인수를 사용하며, 이는 속성 집합입니다.

```nix
# 단일 인수, 속성 집합
# vvvvvvvvvvvvvvvvvv
  { lib, pkgs, ... }: #...
#                   ^ :는 함수임을 의미합니다.
```

이전 설명에서 이 인수는 `lib`와 `pkgs`를 포함하는 속성 집합이어야 한다는 것도 기억하십시오. `...`은 다른 인수를 받을 수 있으며 괜찮다는 것을 의미하며, 우리는 ~~끔찍한 사람들이기 때문에~~ 효율적인 고급 사용자이기 때문에 신경 쓰지 않습니다.

`lib`와 `pkgs`는 어디서 오는 걸까요? 모듈 시스템에서 제공됩니다. 나중에 사용할 `config`도 볼 수 있습니다. [자신만의 특별한 인수를 전달](https://discourse.nixos.org/t/pass-specialargs-to-the-home-manager-module/33068)할 수도 있지만, 이는 이 가이드의 범위를 벗어납니다.

자, 함수입니다. 무엇을 반환할까요? "또 다른 속성 집합"이라고 말했다면 Nix에 익숙해지고 있는 것입니다! 항상 또 다른 속성 집합입니다.

속성 집합에는 무엇이 있습니까? 최상위 `home` 속성이 있으며, 여기에는 `packages`, `username`, `homeDirectory`, `stateVersion`이 포함됩니다. 잠시 후에 이것들을 살펴보겠습니다.

모듈은 기본적으로 두 가지 "모드"로 실행됩니다. 첫 번째는 최상위 속성이 여기에서처럼 구성에 사용되는 값인 더 간단한 방법입니다. 다른 하나는 최상위 속성을 `config`에 할당된 새 속성 집합으로 분리한 다음 전체 모듈을 수정하는 데 사용할 수 있는 일부 `options` 정의를 제공하는 것입니다. 작성하게 될 대부분의 모듈은 아마도 두 번째 방법일 것이며, 곧 다루겠지만, 더 간단하고 야생에서 `config`가 어디에 있는지 궁금해하는 대신 인식할 수 있도록 이 첫 번째 방법을 먼저 보여주고 싶었습니다.

따라서 기본적으로 모듈은 함수를 포함하는 파일입니다. 해당 함수는 (모든 Nix 함수와 마찬가지로) 단일 인수를 사용합니다. 인수는 모듈 시스템에서 제공하는 여러 입력을 포함하는 속성 집합입니다. 출력은 일부 구성 값을 포함하는 속성 집합입니다.

더 많은 정보와 훨씬 더 깊이 있는 내용은 다음을 참조하십시오.

- [모듈에 대한 NixOS 설명서](https://nixos.org/manual/nixos/stable/#sec-writing-modules)
- [모듈에 대한 nix.dev 자습서](https://nix.dev/tutorials/module-system/module-system.html)

## `with` 키워드

이 파일에는 이전에 보지 못한 새로운 것이 하나 있습니다. 바로 `with` 키워드입니다.

```nix
{ lib, pkgs, ... }: {
  home = {
    # ... 기타 항목 ...
    packages = with pkgs; [
      hello
      cowsay
    ];
  };
}
```

간단히 말해서 `with`는 다음 표현식에서 사용되는 변수가 이미 선언되지 않은 경우 `with`에 있는 것을 확인하도록 Nix에 지시합니다.

더 간단한 버전으로, 위와 동일한 것을 `with` 없이 작성하는 또 다른 방법은 다음과 같습니다.

```nix
{ lib, pkgs, ... }: {
  home = {
    # ... 기타 항목 ...
    packages = [
      pkgs.hello
      pkgs.cowsay
    ];
  };
}
```

표현식에 `something.x`와 `something.y`가 많은 경우 이것이 얼마나 유용한지 알 수 있기를 바랍니다. 자, 드디어 좋은 내용으로 넘어갑니다.

## home.nix에는 무엇이 들어갈 수 있습니까?

여기까지 오신 것을 축하합니다. 여기서부터 재미가 시작됩니다. [MyNixOS의 새로운 사탕 가게에 오신 것을 환영합니다](https://mynixos.com/home-manager/options).

위 링크는 Home Manager에서 사용할 수 있는 모든 최상위 옵션에 대한 것입니다. 목록을 보면 [`home`](https://mynixos.com/home-manager/options/home)이 표시됩니다. 그런 다음 그것을 따라가면 `home` 속성에서 지정할 수 있는 모든 옵션을 볼 수 있습니다. 이 목록을 살펴보면 [`packages`](https://mynixos.com/home-manager/option/home.packages)가 표시됩니다.

잠시 시간을 내어 이 사이트를 탐색하여 설정한 다른 `home` 값인 `username`, `homeDirectory`, `stateVersion`을 찾아보십시오. 기다리겠습니다.

혼란스러운 것은 아마도 `stateVersion`일 것입니다. 이것은 잠시 동안 약간 마법 같을 수 있지만 기본 아이디어는 도구가 해석 방법을 알 수 있도록 전체 설정을 특정 스키마에 고정하는 것입니다. 이렇게 하면 이전 설정을 손상시키지 않고 전체 시스템에 새로운 변경 사항을 적용할 수 있습니다. 실용적인 일상 사용자를 위한 간단한 버전: 설정한 다음 다시는 변경하거나 보지 마십시오.

다른 것을 찾았습니까? 좋습니다. 우리가 할 수 있는 다른 재미있는 일을 살펴보겠습니다.

## 홈 디렉터리에 파일 추가

기존 설정에서 아무것도 손상시키지 않아야 하지만 이것이 어디로 가는지에 대한 잠재력을 보여줄 간단한 것부터 시작하겠습니다. 파일을 추가해 봅시다... _파일_입니다. 혁신입니다.

시작하기 전에 한 가지 분명히 하고 싶습니다. 장기적으로는 실제로 이것을 자주 사용하지 않을 것입니다. 구성 파일을 생성하는 특정 구성 옵션을 사용하거나 셸 스크립트 등을 빌드하는 다른 기능을 사용하는 것을 선호하게 될 것입니다. 그러나 이것으로 시작하기는 매우 쉽고 일부 아이디어를 스케치하거나 원하는 곳에 파일을 가져오는 데 유용할 수 있습니다. 대부분은 더 많은 새로운 구문을 보여줄 좋은 변명입니다.

### 우선 파일 가져오기

먼저 [여기 설명서를 확인](https://mynixos.com/home-manager/options/home.file.%3Cname%3E)하여 우리가 하려는 작업을 참조하십시오. 구체적으로 시작하려면 `text` 필드에 관심이 있습니다. 복잡한 파일을 복사하기 위해 `source`를 사용할 수도 있지만 더 간단하고 잠시 후에 더 흥미로운 템플릿을 시작할 수 있으므로 `text`를 사용할 것입니다.

좋은 내용으로 건너뛰고 작동하는 시작점을 제공하겠습니다. 그런 다음 이것을 더 흥미롭게 만들 것입니다.

```nix
{ lib, pkgs, ... }: {
  home = {
    # ... 기타 항목 ...
    file = {
      "hello.txt".text = "Hello, world!";
    };
  };
}
```

이 `"hello.txt"`가 설명서에서 `<name>` 역할을 한다는 점에 유의하십시오. 또한 다음과 같이 할 수도 있다는 것을 기억하십시오.

```nix
"hello.txt" = {
    text = "Hello, world!";
};
```

이 변경 사항을 적용하면 이제 다음을 수행할 수 있습니다.

```bash
cat ~/hello.txt
```

그리고 `Hello, world!` 텍스트가 표시되어야 합니다!

그렇다면 이 파일은 무엇일까요? `ls -l ~/hello.txt`를 수행하면 `/nix/store`의 파일에 대한 링크이며 읽기 전용임을 알 수 있습니다. 홈 매니저 설정을 편집하지 않는 한 이 파일을 편집할 수 없습니다(또는 적어도 편집해서는 안 됩니다).

축하합니다. 텍스트 파일을 만들었습니다. 이제 _흥미로운_ 텍스트 파일로 만들어 봅시다.

### 파일을 스크립트로 만들기

그렇다면 흥미로운 텍스트 파일이란 무엇일까요? 우리를 맞이하는 스크립트는 어떻습니까? [`executable` 옵션](https://mynixos.com/home-manager/option/home.file.%3Cname%3E.executable)을 사용하여 실행할 수 있는 것으로 만들어 봅시다.

```nix
{ lib, pkgs, ... }: {
  home = {
    # ... 기타 항목 ...
    file = {
      "hello.txt" = {
        text = "echo 'Hello, world!'";
        executable = true;
      };
    };
  };
}
```

_다시 말하지만, 이것은 실제로 자신만의 유틸리티 스크립트를 작성하기 위한 장기적인 해결책이 아닙니다. 원하는 장기적인 해결책은 아마도 nixpkgs의 [`writeShellApplication`](https://ryantm.github.io/nixpkgs/builders/trivial-builders/#trivial-builder-writeShellApplication)일 것이며, 이는 `pkgs.writeShellApplication`에서 사용할 수 있습니다. 이를 위해서는 상당한 추가 작업과 이해가 필요하므로 나중에 즐겨찾기에 추가하고 일반적으로 Nix에 더 익숙해지면 다시 돌아오십시오. 이것이 어떤 모습일지 맛보고 싶다면 [자신의 설정에서 유용한 래퍼 스크립트를 많이 생성하는 방법은 다음과 같습니다](https://github.com/Evertras/nix-systems/blob/main/home/modules/shell/funcs/default.nix)._

이를 염두에 두고 변경 사항을 적용하고 우리가 한 일을 살펴보겠습니다.

```bash
# 이제 실행 가능으로 표시됩니다.
ls -l ~/hello.txt

# 예상대로 직접 실행할 수 있습니다!
~/hello.txt
```

### 여러 줄 문자열

자, 멋지네요... 하지만 단일 명령보다 더 많은 명령을 실행하고 싶다면 어떻게 해야 할까요? 스크립트에서 새 줄을 어떻게 처리할까요? [`''` 표기법을 사용합니다](https://nixos.org/manual/nix/stable/language/values.html#type-string)!

```nix
{ lib, pkgs, ... }: {
  home = {
    # ... 기타 항목 ...
    file = {
      "hello.txt" = {
        text = ''
          #!/usr/bin/env bash

          echo "Hello, world!"
          echo '*slaps roof* 이 스크립트에는 많은 줄을 넣을 수 있습니다.'
        '';
        executable = true;
      };
    };
  };
}
```

`''` 표기법을 사용하면 원하는 만큼 많은 줄을 작성할 수 있으며 예상대로 모든 것을 이동하기 위해 가장 왼쪽 들여쓰기와 일치합니다.

다시 한 번 변경 사항을 적용하고 실행하여 두 줄을 모두 에코하는지 확인하십시오. `cat ~/hello.txt`를 수행하여 모든 것이 있는지 확인할 수도 있습니다.

### 변수 및 문자열 보간 사용

Nix가 단순한 미화된 JSON이 아니라고 말한 것을 기억하십니까? 스크립트에 더 동적인 것을 추가하여 증명해 봅시다.

먼저 사용자 이름을 변수로 추출해 보겠습니다. `let in` 표기법을 생각했다면 잘했습니다!

여기에 있는 동안 [홈 디렉터리](https://mynixos.com/home-manager/option/home.homeDirectory)에 사용자 이름이 하드코딩되어 있음을 알 수 있습니다. 또한 [`homeDirectory` 변수](https://mynixos.com/home-manager/option/home.homeDirectory)를 명시적으로 설정할 필요가 없을 수도 있지만 문자열 내에서 변수를 사용하는 방법을 살펴볼 좋은 변명이라는 것도 알 수 있습니다.

기본적으로 `"my string has ${thing} in it"`은 `thing`의 값을 문자열에 주입합니다. `"my string has " + thing + " in it"`과 같이 간단한 문자열 결합도 수행할 수 있습니다.

홈 디렉터리에 이것을 어떻게 사용할 수 있는지 살펴보겠습니다.

```nix
# 모든 표현식 앞에 'let in'을 사용할 수 있지만 사용자 이름을
# 높은 수준에서 사용할 수 있도록 여기에 넣겠습니다.
{ lib, pkgs, ... }: let
  username = "myusername";
in {
  home = {
    # ... 기타 항목 ...
    # 'inherit'을 피하고 있다고 생각했습니까? 돌아왔습니다... 기억하십시오.
    # 이것은 단지 "username = username;"입니다.
    inherit username;

    # 홈 디렉터리에 사용자 이름 주입
    homeDirectory = "/home/${username}";

    # 이것도 할 수 있지만 직접 보간을 사용하는 것이
    # 복잡한 작업을 수행하면 훨씬 깔끔해 보입니다.
    # homeDirectory = "/home/" + username;
  };
}
```

이것은 우리 스크립트에도 적용됩니다.

```nix
{ lib, pkgs, ... }: let
  username = "myusername";
in {
  home = {
    # ... 기타 항목 ...
    file = {
      "hello.txt" = {
        text = ''
          #!/usr/bin/env bash

          echo "Hello, ${username}!"
          echo '*slaps roof* 이 스크립트에는 많은 줄을 넣을 수 있습니다.'
        '';
        executable = true;
      };
    };
  };
}
```

이 시점에서 _아이디어_를 얻기 시작해야 합니다. 모든 구성 파일, bashrc, zshrc, vimrc, emacs 구성, gitconfig, ~~세금 신고서~~, 계속 수정하고 모든 곳에 복사해야 하는 모든 것을 생각해 보십시오. 이제 해당 파일을 생성하고, 유형 안전하고 재현 가능한 방식으로 변경하기 위한 입력을 제공하여 새 VM을 시작하고 `make`를 실행하기만 하면 모든 것이 원하는 대로 정확하게 되도록 하는 것을 상상해 보십시오.

결국 `home.file`로 수행하지 않을 수도 있지만 Nix의 강력함을 느끼기 시작해야 합니다!

## 여기서부터 어디로 가야 할까요?

이 시점에서 '모듈'이 무엇인지, 일부 홈 매니저 옵션을 어디서 찾을 수 있는지, 어떻게 설정하는지에 대한 기본적인 이해가 있어야 합니다. 또한 `with`, 여러 줄 문자열 및 문자열 보간을 사용하는 새로운 Nix 구문을 배웠습니다.

시간이 나면 구성을 여러 모듈로 분할하는 방법에 대한 섹션을 작성하고 싶습니다. 미리 시작하고 싶다면 `imports` 속성을 확인하고 모듈이 다른 모듈을 가져올 수 있고 다른 모듈이 다른 모듈을 가져올 수 있다는 것을 깨달으십시오.

`home.file`에 모든 것을 작성하기 전에 Nix에는 아마도 하려는 작업을 수행하는 더 나은 방법이 거의 항상 내장되어 있다는 것을 기억하십시오. 예를 들어 [bash](https://mynixos.com/home-manager/options/programs.bash)를 확인하고 Home Manager가 bashrc를 관리하도록 활성화해 보십시오. bash 별칭, 함수 및 환경 변수와 같은 것을 추가하기 시작할 수 있습니다.

중요한 것은 이 시점에서 작동하는 Home Manager 설정과 스스로 실험을 시작할 수 있는 기본 능력이 있어야 한다는 것입니다. 시간이 지남에 따라 자신에게 맞는 방식으로 자신만의 설정을 개발하고 구축하고 유지 관리할 수 있게 될 것입니다.

Nix를 즐기십시오!
