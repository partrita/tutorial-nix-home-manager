# 02 - 간단한 Flake로 기본 리포지토리 만들기

[<- 01-install_ko.md](./01-install_ko.md) | [03 - 설명: 기본 Nix 구문 및 flake.nix 입력 ->](./03-explain-inputs_ko.md)


## Flake란 무엇인가?

음, 지금은 신경 쓰지 말고 계속 진행하세요. 나중에 자세히 알아볼 겁니다.


## Git 저장소 초기화하기

Home Manager 설정을 Git 저장소로 만들면 여러모로 편리합니다. 재현 가능하고 버전 관리가 되며, 다른 시스템으로 옮기기도 쉽기 때문이죠.

```bash
# 원하는 이름으로 만드세요. 어떤 이름이든 괜찮습니다!
mkdir my-home-manager
cd my-home-manager
git init
```

## `flake.nix` 만들기

이제 간단한 Flake를 만들어 봅시다. 이 필드들이 왜 이렇게 구성되어 있는지, 입력과 출력은 무엇인지 궁금할 겁니다. 하지만 지금은 그저 이대로 따라 해주세요. 이 파일은 여러 사용자 프로필을 정의하고 해당 설정을 불러오는 방법을 담고 있다는 것만 알아두시면 됩니다.

답을 알고 싶다는 걸 압니다. 저 역시 그랬으니까요. 예전에 이 부분을 접했을 때, 계속 진행하기 전에 왜 이런 작업을 하는지 파고들다가 좌절했던 기억이 납니다. 그건 실수였어요. 저를 믿고 일단 **작동하는 무언가를 먼저 만드세요.** 그러면 직접 이것저것 만져보면서 궁금증을 해소할 수 있게 될 겁니다. 지금은 너무 깊은 토끼굴이에요. 답은 다음 섹션에서 나올 겁니다!

그래도 이건 꼭 지켜주세요. **직접 입력하세요.** 복사/붙여넣기하지 마세요. 답답하겠지만, 길게 돌아가는 것이 기억에 더 오래 남습니다. 뇌 과학적인 이유가 있어요. 그냥 하세요.

```nix
# flake.nix
# 복사/붙여넣기 하지 마세요. 혹시 건너뛰고 싶었다면 위 내용을 먼저 읽으세요.
{
  description = "내 Home Manager 설정";

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
      system = "x86_64-linux"; # 여러분의 시스템 아키텍처에 맞게 변경해야 할 수도 있습니다.
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

이 코드를 입력하면서 (복사/붙여넣기 **아니고**요, 속이지 마세요!) `myprofile`을 보셨을 겁니다. 이것을 **실제 리눅스 사용자 이름**, 혹은 강아지 이름, 좋아하는 행성 이름 등 아무거나 바꾸세요. 영숫자(`[a-z0-9-]+`)이기만 하면 됩니다. 나중에 기억해야 하니 잘 정해주세요. 물론, 원한다면 지금은 `myprofile` 그대로 두어도 괜찮습니다.



## `home.nix` 만들기

위에서 `./home.nix` 참조를 보셨을 겁니다. 이제 저장소 루트에도 이 파일을 만들어 봅시다.

다시 한번 강조하지만, 직접 입력하세요. 아직 설명은 없을 겁니다. 거의 다 왔어요. 답이 곧 나옵니다!

```nix
{ lib, pkgs, ... }:
{
  home = {
    packages = with pkgs; [
      hello
    ];

    # 이 부분을 실제 사용자 이름으로 설정해야 합니다.
    username = "myusername";
    homeDirectory = "/home/myusername";

    # 나중에 이 가이드를 다시 보더라도 변경할 필요가 없습니다.
    # 첫 빌드 후에는 절대로 변경하지 마세요. 이유는 묻지 마세요.
    stateVersion = "23.11";
  };
}
```

이번에는 `username`과 `homeDirectory` 모두 **실제 사용자 이름**과 일치시켜야 합니다.



## 체크포인트

계속 진행하기 전에 다음 명령어를 실행해 보세요.

```bash
# 셸에서 이걸 실행해 보세요.
hello
```

`hello`를 찾을 수 없다는 오류가 표시된다면 잘 된 겁니다! 이제 Home Manager가 제대로 작동하는지 확인할 수 있습니다.



## `Makefile` 만들기

매번 명령에 긴 플래그를 입력하는 건 정말 귀찮죠. 그래서 실행 전에 모든 작업을 처리해 줄 `Makefile`을 만들어 봅시다.

```make
# 복사/붙여넣기 할 때 주의하세요! Makefile은 탭을 원합니다!
# ...하지만 여러분은 복사/붙여넣기를 안 하고 있죠, 그렇죠?
.PHONY: update
update:
	home-manager switch --flake .#myprofile
```

`myprofile` 부분을 `flake.nix`에서 변경한 이름으로 바꿔주세요. 이것이 나중에 다른 프로필을 대상으로 지정하는 방법입니다.

만약 `Makefile`을 건너뛰고 매번 명령어를 직접 입력하거나 다른 방식으로 실행하고 싶다면 그래도 괜찮습니다. 저는 그냥 `Makefile`을 좋아할 뿐이에요.



## Home Manager 활성화하기

### Flake의 첫 난관

```bash
# 이 명령은 오류를 낼 겁니다. 아마 무슨 말인지 이해가 안 될 거예요. 아래를 읽어보세요.
make
```

여러분들이 너무 앞서가지 않고 제가 요청한 작업만 정확히 수행했다고 가정하면, 파일이 없다는 오류가 보일 겁니다. 하지만 파일은 분명히 거기에 있죠. 저도 알고 여러분도 압니다. 그럼 왜 Nix는 모르는 걸까요?

그 이유는 첫 번째 설명이 여러분을 혼란스럽게 할 것이기 때문입니다. Flakes가 있는 Nix는 Git 저장소에 추가되지 않은 모든 것을 **완전히 무시합니다.** 사실 이건 좋은 점이에요. 약속합니다! 모든 것이 Git 저장소 안에 있어야만 모든 것이 재현 가능하도록 보장됩니다. 그러니 문제를 해결하려면 모든 파일을 Git에 추가하기만 하면 됩니다. 만약 이미 습관적으로 이 작업을 했다면, 이미 모든 것이 잘 작동하는 것을 보셨을 수도 있습니다.

### 이제 정말로 활성화하기

```bash
# 모든 것을 Git에 추가합니다.
git add flake.nix home.nix Makefile
# 기술적으로는 커밋할 필요는 없고, 추가만 해도 되지만... 그래도
git commit -m "첫 home-manager 커밋"

# 이제 작동해야 합니다.
make

# 이제 'hello'를 실행할 수 있을 겁니다!
hello

# 그리고 nix 저장소 경로를 확인할 수 있습니다.
which hello

# 이제 flake.lock 파일도 생겼네요. 뭘까요? 나중에 알아보세요.
git add flake.lock
git commit -m "flake.lock 추가"
```

만약 `hello`를 찾을 수 없다면, 설치 단계를 제대로 따랐는지 다시 확인하세요. 특히 `.bashrc` 또는 그에 준하는 파일에서 소싱하라고 지시한 파일을 제대로 소싱했는지 말이죠!

`hello`를 찾았고 출력이 보인다면, 축하합니다! 이제 Flakes와 함께 Home Manager를 사용하고 있는 겁니다. 바라건대 저보다 훨씬 빨리 해냈기를 바랍니다.



## 첫 번째 수정

원한다면 [NixOS 패키지](https://search.nixos.org/packages)에서 다른 패키지들을 추가할 수 있습니다. 몇 가지를 추가하고 `make`를 다시 실행하면 셸에서 해당 패키지에 접근할 수 있게 됩니다. 패키지를 추가하려면 그냥 이름을 공백으로 구분해서 나열하기만 하면 됩니다. 이렇게요.

```nix
{ lib, pkgs, ... }:
{
  home = {
    packages = with pkgs; [
      hello
      # 새 줄에 있든 없든 상관없습니다.
      # 공백으로만 구분하면 됩니다.
      cowsay lolcat
    ];

    # 이 부분은 실제 사용자 이름으로 설정해야 합니다.
    username = "myusername";
    homeDirectory = "/home/myusername";

    # 첫 빌드 후에는 절대로 변경하지 마세요. 이유는 묻지 마세요.
    stateVersion = "23.11";
  };
}
```

패키지를 제거할 수도 있습니다. `home.nix`에서 `hello`를 제거하고, 적용 후에 셸에서 `hello`를 더 이상 실행할 수 없는지 확인해 보세요.

**이것이 바로 Home Manager 사용의 강력한 장점 중 하나입니다.** `apt install cowsay`처럼 시스템에 영원히 찌꺼기를 남기는 대신, 원하는 것만 명시적으로 나열하는 거죠. 대부분의 패키지 관리자에서 발생하는 느린 찌꺼기 축적을 방지하여 정말 좋습니다!

덤으로, Nix 리스트에 대해서도 방금 배웠네요! Nix 리스트는 공백으로 구분된 항목들일 뿐입니다. 공백의 양은 중요하지 않아요. 쉼표는 좋아하지 않습니다.



## 정리 단계

끝내기 전에 `Makefile`에 정리 작업을 추가해야 합니다. 이렇게 하면 Nix가 장기간 많은 변경 사항으로 인해 디스크 공간을 너무 많이 차지하지 않도록 할 수 있습니다.

```make
# 복사/붙여넣기 할 때 주의하세요! Makefile은 탭을 원합니다!
.PHONY: update
update:
	home-manager switch --flake .#myusername

.PHONY: clean
clean:
	nix-collect-garbage -d
```

Nix는 롤백을 위해 의도적으로 이전 세대를 유지하기 때문에, 지시가 있을 때만 스스로 정리합니다. 지금은 너무 걱정하지 마세요. 가끔 `make clean`을 실행해야 한다는 것만 알아두면 됩니다. 이 특정 명령은 기록을 위해 이전 세대를 제거합니다. 원할 때마다 실행할 수 있지만, 다음 실행 시 일부 종속성을 다시 다운로드하게 될 겁니다.



## 문제 해결

여전히 문제가 있다면, 현재 파일들을 [./02-basic/](./02-basic/)에 있는 파일과 비교하여 누락된 것이 없는지 확인해 보세요.


## 요약

우리는 다음을 추가했습니다.

  * 사용자 프로필을 정의하는 `flake.nix` 파일
  * 사용자 프로필의 설정을 정의하는 `home.nix` 파일
  * 적용 과정을 쉽게 만들어 주는 `Makefile`

이제 사용자 프로필에 패키지를 추가하고 제거할 수 있습니다.

그리고 `make clean`으로 시스템을 정리할 수 있죠.



## 다음 단계

여기서부터 Nix의 세계가 열리고, `home.nix` 파일을 수정하여 온갖 재미있는 일들을 시작할 수 있습니다.

나중에는 [bashrc 관리](https://mynixos.com/home-manager/options/programs.bash)와 같은 작업을 위한 흥미롭고 유용한 설정 옵션을 찾는 방법, `home.nix`를 자체 설정 옵션으로 별도의 모듈 파일로 구성하는 방법 등에 대해 이야기할 겁니다.

하지만 일단은 [다음 섹션에서 몇 가지 추가 설명을 해야합니다.](./03-explain-inputs_ko.md)
