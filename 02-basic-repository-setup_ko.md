# 02 - 간단한 Flake로 기본 리포지토리 만들기

[<- 01-install_ko.md](./01-install_ko.md) | [03 - 설명: 기본 Nix 구문 및 flake.nix 입력 ->](./03-explain-inputs_ko.md)

## Flake란 무엇인가-

신경 쓰지 말고 계속 진행하고 나중에 알아보십시오.

## Git 리포지토리 초기화

Home Manager 구성을 위한 Git 리포지토리를 만들고 싶을 것입니다. 재현 가능하고 버전 관리되며 시스템 간에 쉽게 전송할 수 있기 때문입니다.

```bash
# 원하는 대로 이름을 지정하십시오. 판단하지 않겠습니다.
mkdir my-home-manager
cd my-home-manager
git init
```

## flake.nix 만들기

간단한 Flake를 만들어 봅시다. 왜 이 필드들이 이런 식으로 되어 있을까요? 입력은 무엇이고 출력은 무엇일까요? 걱정하지 말고 지금은 그냥 이렇게 하십시오. 지금 알아야 할 유일한 것은 이 파일이 여러 사용자 프로필을 정의하고 해당 구성을 로드하는 방법에 대한 정보를 포함한다는 것입니다.

답을 원한다는 것을 압니다. 저도 답을 원했습니다. 이전에 보았을 때 계속 진행하기 전에 왜 이런 일을 하는지 파헤치려고 애쓰면서 좌절했습니다. 그것은 실수였습니다. 저를 믿으십시오. 그냥 밀어붙여서 먼저 작동하는 무언가를 만드십시오. 그러면 가지고 놀 수 있기 때문에 답을 얻을 수 있습니다. 지금은 토끼굴이 너무 깊습니다. 다음 섹션에서 답이 나올 것입니다!

그렇긴 하지만 직접 입력하십시오. 복사/붙여넣기하지 마십시오. 짜증 날 것입니다. 하지만 긴 길을 가는 것이 기억하는 데 도움이 될 것입니다. 신경과학적인 문제입니다. 그냥 하십시오.

```nix
# flake.nix
# 복사하여 붙여넣지 마십시오. 속이려고 훑어보았다면 위를 먼저 읽으십시오.
{
  description = "내 Home Manager 구성";

  inputs = {
    nixpkgs.url = "nixpkgs/nixos-23.11";

    home-manager = {
      url = "github:nix-community/home-manager/release-23.11";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

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

이것을 입력할 때(그리고 복사/붙여넣기가 **아니라**, 속이지 마십시오) `myprofile`을 보셨습니까? 실제 Linux 사용자 이름, 강아지 이름 또는 좋아하는 행성으로 변경하십시오. 영숫자(`[a-z0-9-]+`)이기만 하면 상관없습니다. 나중에 기억하십시오. 정말 원한다면 지금은 `myprofile`로 남겨둘 수도 있습니다.

## home.nix 만들기

위에서 `./home.nix` 참조를 보셨을 것입니다. 리포지토리 루트에도 해당 파일을 만들어 봅시다.

다시 말하지만 직접 입력하십시오. 다시 말하지만 아직 이에 대한 설명은 없을 것입니다. 거의 다 왔습니다. 답이 오고 있습니다.

```nix
{ lib, pkgs, ... }:
{
  home = {
    packages = with pkgs; [
      hello
    ];

    # 이것은 실제로 사용자 이름으로 설정해야 합니다.
    username = "myusername";
    homeDirectory = "/home/myusername";

    # 나중에 이 글을 읽는 경우 변경할 필요가 없습니다.
    # 첫 번째 빌드 후에는 절대 변경하지 마십시오. 질문하지 마십시오.
    stateVersion = "23.11";
  };
}
```

이번에는 실제로 사용자 이름을 `username`과 `homeDirectory` 모두에 일치시켜야 합니다.

## 체크포인트

계속 진행하기 전에 이것을 시도해 보십시오.

```bash
# 셸에서 이것을 실행해 보십시오.
hello
```

`hello`를 찾을 수 없다는 오류가 표시되면 좋습니다! 이제 Home Manager가 제대로 작동하는지 확인할 수 있습니다.

## Makefile 만들기

명령에 플래그를 반복해서 입력하는 것이 싫습니다. 그래서 실행하기 전에 Makefile이 모든 작업을 수행하도록 만들어 봅시다.

```make
# 복사/붙여넣기에 주의하십시오. Makefile은 탭을 원합니다!
# 하지만 복사/붙여넣기를 하고 있지 않죠?
.PHONY: update
update:
    home-manager switch --flake .#myprofile
```

`myprofile`을 `flake.nix`에서 변경한 것으로 변경하십시오. 이것이 나중에 다른 프로필을 대상으로 지정하는 방법입니다.

Makefile을 건너뛰고 매번 명령을 입력하거나 다른 방법으로 명령을 실행하려는 경우 괜찮습니다. 저는 그냥 Makefile을 좋아합니다.

## home-manager 활성화

### 먼저 레이크에 걸려 넘어지기

```bash
# 이것은 오류가 발생합니다. 아무 의미도 없을 것입니다. 아래를 읽으십시오.
make
```

여러분이 과도하게 성취하지 않고 제가 요청한 작업만 수행했다고 가정하면 파일이 없다는 오류가 표시될 것입니다. 하지만 파일은 거기에 있습니다. 저는 그것을 압니다. 여러분도 그것을 압니다. 왜 Nix는 그것을 모를까요?

첫 번째 설명은 여러분을 괴롭힐 것이기 때문입니다. Flakes가 있는 Nix는 Git 리포지토리에 추가되지 않은 모든 것을 **완전히 무시**합니다. 이것은 실제로 좋은 일입니다. 약속합니다. 모든 것이 Git 리포지토리에 있어야 하므로 모든 것이 재현 가능하도록 보장됩니다. 따라서 수정하려면 모든 것을 Git에 추가하기만 하면 됩니다. 습관적으로 이미 이 작업을 수행했다면 이미 모든 것이 작동하는 것을 보았을 수도 있습니다.

### 실제로 활성화하기

```bash
# 모든 것을 Git에 추가
git add flake.nix home.nix Makefile
# 기술적으로 커밋할 필요는 없으며 추가만 하면 되지만 여전히
git commit -m "첫 번째 home-manager 커밋"

# 이제 작동해야 합니다.
make

# 이제 hello를 실행할 수 있어야 합니다!
hello

# 그리고 소스로 nix 저장소 경로를 볼 수 있습니다.
which hello

# 이제 flake.lock 파일도 있습니다... 무엇일까요? 나중에 알아보십시오.
git add flake.lock
git commit -m "flake.lock 추가"
```

`hello`를 찾을 수 없는 경우 설치 단계를 올바르게 따랐는지 확인하십시오. 특히 .bashrc 또는 이와 동등한 파일에서 소싱하라고 지시한 파일을 소싱하고 있는지 확인하십시오!

`hello`를 찾았고 출력이 표시되면 축하합니다! Flakes가 있는 Home Manager가 있으며 바라건대 저보다 훨씬 빨리 해냈을 것입니다.

## 첫 번째 수정

원하는 경우 [NixOS 패키지](https://search.nixos.org/packages)에서 더 많은 패키지를 추가하기 시작할 수 있습니다. 몇 개를 추가하고 `make`를 다시 실행하여 셸에서 액세스하십시오. 패키지를 추가하려면 공백으로 이름을 추가하기만 하면 됩니다. 다음과 같이 하십시오.

```nix
{ lib, pkgs, ... }:
{
  home = {
    packages = with pkgs; [
      hello
      # 새 줄에 있든 없든 상관없습니다.
      # 공백만 있으면 됩니다.
      cowsay lolcat
    ];

    # 이것은 실제로 사용자 이름으로 설정해야 합니다.
    username = "myusername";
    homeDirectory = "/home/myusername";

    # 첫 번째 빌드 후에는 절대 변경하지 마십시오. 질문하지 마십시오.
    stateVersion = "23.11";
  };
}
```

패키지를 제거할 수도 있습니다. `home.nix`에서 `hello`를 제거하고 적용 후 셸에서 `hello`를 더 이상 실행할 수 없다는 것을 확인하십시오.

**이것은 실제로 home-manager 사용의 더 강력한 점 중 하나입니다.** `apt install cowsay`를 수행하고 해당 항목을 시스템에 영원히 두는 것과 달리 원하는 것과 원하는 것만 명시적으로 나열합니다. 대부분의 패키지 관리자에서 발생하는 느린 찌꺼기 축적을 방지하여 훌륭합니다!

Nix 목록에 대해서도 방금 배웠습니다! Nix 목록은 공백으로 구분된 항목일 뿐입니다. 어떤 양의 공백이든 괜찮습니다. 쉼표를 좋아하지 않습니다.

## 정리 단계

끝나기 전에 Makefile에 정리 작업을 추가해야 합니다. 이렇게 하면 Nix가 장기간 많은 변경 사항으로 인해 디스크를 잡아먹지 않도록 할 수 있습니다.

```make
# 복사/붙여넣기에 주의하십시오. Makefile은 탭을 원합니다!
.PHONY: update
update:
    home-manager switch --flake .#myusername

.PHONY: clean
clean:
    nix-collect-garbage -d
```

Nix는 롤백을 위해 의도적으로 이전 세대를 유지하기 때문에 지시가 있을 때만 자체적으로 정리합니다. 지금은 걱정하지 마십시오. 가끔 `make clean`을 실행해야 한다는 것만 알아두십시오. 이 특정 명령은 기록을 위해 이전 세대를 제거합니다. 원할 때마다 실행할 수 있지만 다음 실행 시 일부 종속성을 다시 다운로드하게 됩니다.

## 문제 해결

여전히 문제가 있는 경우 파일을 [./02-basic/](./02-basic/)에 있는 것과 비교하고 누락된 것이 없는지 확인하십시오.

## 요약

사용자 프로필을 정의하는 flake.nix 파일, 사용자 프로필의 구성을 정의하는 home.nix 파일, 적용하기 쉽게 만드는 Makefile을 추가했습니다.

이제 사용자 프로필에 패키지를 추가하고 제거할 수 있습니다.

`make clean`으로 정리할 수 있습니다.

## 다음 단계

여기서부터 Nix 세계가 열리고 `home.nix`를 수정하여 온갖 재미있는 일을 시작할 수 있습니다.

나중에는 [bashrc 관리](https://mynixos.com/home-manager/options/programs.bash)와 같은 작업을 수행하기 위한 흥미롭고 유용한 구성 옵션을 찾는 방법, 자체 구성 옵션으로 `home.nix`를 별도의 모듈 파일로 구성하는 방법 등에 대해 이야기할 것입니다.

하지만 지금은 [다음 섹션에서 몇 가지 설명을 해 드려야 합니다.](./03-explain-inputs_ko.md)
