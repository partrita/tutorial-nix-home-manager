# 01 - Nix 및 Home Manager 설치

[<- README_ko.md](README_ko.md) | [02 - 기본 리포지토리 설정 ->](02-basic-repository-setup_ko.md)

## Nix 설치하기

먼저 **Nix**를 설치해야 합니다. 다음 링크에서 설치 가이드를 확인하세요: [Nix 설치](https://nixos.org/download.html).

만약 `root` 권한으로 무언가를 실행하는 것이 꺼림칙하지 않다면, 아마도 **다중 사용자 설치**를 선택하는 것이 좋습니다.

지금 당장은 다른 문서를 파고들고 싶은 유혹이 들겠지만, 참으세요! 그건 나중을 위해 아껴둡시다.

### Flakes 활성화하기

**Flakes**를 활성화해야 합니다. 왜냐고요? 그 이유는 나중에 설명해 드릴게요.

지금은 `~/.config/nix/nix.conf` 파일이나 [Nix 설정 파일이 있는 곳](https://nixos.wiki/wiki/Flakes)에 다음 내용을 추가하세요.

```
experimental-features = nix-command flakes
```

Flakes는 아직 **안정적인 기능이 아니라는 점**을 알아두세요. 하지만 개인 설정에는 괜찮습니다. 더 자세히 알고 싶다면 [공식 페이지](https://nix.dev/concepts/flakes)를 확인해 보세요. 지금은 그저 이 기능이 훌륭하고 필요하며, 나중에 약간 변경될 수 있다는 점만 기억하면 됩니다.

### 제대로 작동하는지 확인하기

다음 명령어를 실행해 보세요.

```bash
nix run nixpkgs#hello
```

잘 되나요? 좋습니다! 이제 Flakes가 활성화된 Nix가 준비되었습니다.

`nixpkgs` 같은 문법이 궁금할 수도 있지만 지금은 생각하지 마세요. 일단 계속 진행하고 나중에 다시 살펴보면 됩니다.

## Home Manager 설치하기

이제 **Home Manager**를 설치할 차례입니다. 다음 링크에서 설치 가이드를 확인하세요: [Home Manager 설치](https://nix-community.github.io/home-manager/index.xhtml#ch-installation).

이 가이드를 읽고 있다면 **독립 실행형(standalone) 버전**을 원할 겁니다. 만약 NixOS를 사용 중이라면, [NixOS를 통해 패키지로 설치](https://search.nixos.org/packages?show=home-manager&query=home-manager)하세요.

가이드의 **4단계**를 꼭 따랐는지 확인하세요.

```bash
# 이 내용은 .bashrc 또는 사용하는 셸에 적용되어야 합니다.
# 나중에는 Home Manager가 이 작업을 자동으로 처리하게 할 수 있지만 일단은 그냥 진행합니다.
source $HOME/.nix-profile/etc/profile.d/hm-session-vars.sh
```

나머지 문서는 읽지 마세요. "시작하기" 섹션도 건너뛰고, 이유나 설명을 찾으려 하지 마세요. 일단 저와 함께 작동하는 환경을 만드는 데 집중하세요. 완벽하게 작동하는 무언가를 갖게 되면, 그때 돌아가서 더 깊이 파고들 수 있습니다.

### 제대로 작동하는지 확인하기

다음 명령어를 실행해 보세요.

```bash
home-manager --version
```

성공했나요? 좋습니다! 이제 Home Manager가 설치되었습니다.

[다음으로!](02-basic-repository-setup_ko.md)
