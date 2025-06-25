# 01 - Nix 및 Home Manager 설치

[<- README_ko.md](README_ko.md) | [02 - 기본 리포지토리 설정 ->](02-basic-repository-setup_ko.md)

## Nix 설치

[Nix 설치](https://nixos.org/download.html).

루트로 무언가를 실행하는 아이디어가 정말 싫지 않다면 아마도 다중 사용자 설치를 원할 것입니다.

여기서 임의의 문서를 파헤치고 싶을 수도 있습니다. 그러지 마십시오. 그건 다른 날을 위해 남겨두십시오.

### Flakes 활성화

Flakes를 활성화하고 싶을 것입니다. 왜냐고요? 나중에 알아보십시오.

지금은 `~/.config/nix/nix.conf` 또는 [Nix 구성이 있는 곳](https://nixos.wiki/wiki/Flakes)에 다음을 입력하십시오.

```
experimental-features = nix-command flakes
```

Flakes가 안정적이라고 보장되지 않는다는 점은 적어도 알고 있어야 하지만 개인 설정에는 괜찮습니다. 더 자세히 알아볼 준비가 되면 [여기 공식 페이지를 확인하십시오](httpsix.dev/concepts/flakes). 지금은 그냥 훌륭하고 원하며 나중에 약간 변경될 수 있으므로 알고만 있으십시오.

### 작동하는지 확인

이것을 실행할 수 있습니까?

```bash
nix run nixpkgs#hello
```

예? 좋습니다. Flakes가 활성화된 Nix가 있습니다.

그 구문은 무엇이었습니까? `nixpkgs`는 무엇입니까? 걱정하지 말고 계속 진행하고 나중에 살펴보십시오.

## Home Manager 설치

[Home Manager 설치](https://nix-community.github.io/home-manager/index.xhtml#ch-installation).

이 글을 읽고 있다면 독립 실행형을 원할 것입니다. NixOS를 사용하는 경우 [NixOS를 통해 패키지로 설치](https://search.nixos.org/packages?show=home-manager&query=home-manager)하십시오.

4단계를 따랐는지 확인하십시오.

```bash
# 이것은 .bashrc 또는 사용 중인 셸에서 소싱되어야 합니다.
# 나중에는 home-manager가 이 작업을 수행하도록 할 수 있지만 지금은 부트스트래핑 중입니다...
source $HOME/.nix-profile/etc/profile.d/hm-session-vars.sh
```

나머지 문서를 읽지 마십시오. 시작하기 부분을 시도하지 마십시오. 이유나 설명을 찾지 마십시오. 저와 함께 계십시오. 작동하는 무언가가 있으면 돌아갈 수 있습니다.

### 작동하는지 확인

이것을 실행할 수 있습니까?

```bash
home-manager --version
```

예? 좋습니다. Home Manager가 있습니다.

[다음으로!](02-basic-repository-setup_ko.md)
