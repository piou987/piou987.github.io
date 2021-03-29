# Windows setup

나는 주로 Windows에서 작업할 때 `Windows terminal` + `WSL2` 를 쓴다. 세팅하는 방법을 작성해본다.



## WSL2

| URL                                                        | Description |
| ---------------------------------------------------------- | ----------- |
| https://docs.microsoft.com/ko-kr/windows/wsl/install-win10 | 참조        |



## Windows terminal

Windows terminal은 Powershell, WSL, CMD 등의 여러 터미널을 하나의 앱에서 실행할 수 있기 때문에 사용하고 있다. 특히 Customizing이 쉽기 때문에도 사용한다.

#### 1. Install Windows terminal

Microsoft store에서 Windows terminal을 다운로드합니다.

#### 2. install fonts

fonts를 다운로드 받고 `install.ps1`파일을 실행하여 설치합니다.

https://github.com/powerline/fonts -> DownloadZIP -> 압축해제 -> `Windows powershell `관리자 권한으로 실행

```
PS C:\Users\82108\Downloads> cd C:\Users\82108\Downloads     <--- 압축해제한 폴더로 이동
PS C:\Users\82108\Downloads> .\install.ps1
```

만약 아래와 같이 에러가 발생하면 Windows 정책을 변경해줘야 합니다. 

```
C:\Users\82108\Downloads\install.ps1 파일을 로드할 수 없습니다. 자세한 내용은 about_Execution_Policies(https://go.microsoft.com/fwlink/?LinkID=135170)를 참조하십시오. 위치 줄:1 문자:1 + yarn -v + ~~~~ + CategoryInfo : 보안 오류: (:) [], PSSecurityException + FullyQualifiedErrorId : UnauthorizedAccess
```

```
PS C:\Windows\system32> ExecutionPolicy      <-- 현재상태확인
Restricted        <---- 모든 스크립트 막음

PS C:\Windows\system32> Set-ExecutionPolicy Unrestricted

PS C:\Windows\system32> ExecutionPolicy       <-- 다시 확인
Unrestricted     <---- 모든 스크립트 허용으로 바뀐거 확인 됨.
```



폰트 설치가 완료되면 정책을 원래대로 돌려놓습니다.

```
PS C:\Windows\system32> Set-ExecutionPolicy Unrestricted
```

#### 3. settings.json

| URL                                                          | Title |
| ------------------------------------------------------------ | ----- |
| https://blog.nillsf.com/index.php/2020/02/17/setting-up-wsl2-windows-terminal-and-oh-my-zsh/ | 참조  |

Windows terminal 을 실행하고 settings를 들어갑니다.

![img](https://blog.nillsf.com/wp-content/uploads/2020/02/image-41.png)

원하는 에디터를 선택하면 settings.json파일이 열립니다.

```
{
    ... {생략}
    "defaultProfile": "{07b52e3e-de2c-5db4-bd2d-ba144ed6c273}",
    "profiles":
    {
        ...{생략}
        "list":
        [
            ...{생략}
            {
                "guid": "{07b52e3e-de2c-5db4-bd2d-ba144ed6c273}",
                "hidden": false,
                "name": "Ubuntu-20.04",
                "source": "Windows.Terminal.Wsl",
                "colorScheme" : "wsl",
                "fontFace" : "DejaVu Sans Mono for Powerline"
            }
        ]
    },
    
    "schemes": [
        {
            "background" : "#002B36",
            "black" : "#002B36",
            "blue" : "#268BD2",
            "brightBlack" : "#657B83",
            "brightBlue" : "#839496",
            "brightCyan" : "#D33682",
            "brightGreen" : "#B58900",
            "brightPurple" : "#EEE8D5",
            "brightRed" : "#CB4B16",
            "brightWhite" : "#FDF6E3",
            "brightYellow" : "#586E75",
            "cyan" : "#2AA198",
            "foreground" : "#93A1A1",
            "green" : "#859900",
            "name" : "wsl",
            "purple" : "#6C71C4",
            "red" : "#DC322F",
            "white" : "#93A1A1",
            "yellow" : "#B58900"
        }
    ],

    ...{생략}
}

```



## zsh

#### 1. install zsh

```
sudo apt update
sudo apt install -y zsh 
```



#### 2. install oh-my-zsh

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```



#### 3. agnoster theme

`~/.zshrc`

```
# ZSH_THEME="robbyrussell"
ZSH_THEME="agnoster"

# add new line
cd ~
```



`~/.oh-my-zsh/themes/agnoster.zsh-theme`

```
{% raw %} # jekyll template tag

# Context: user@hostname (who am I and where am I)
prompt_context() {
  if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
#    prompt_segment black default "%(!.%{%F{yellow}%}.)%n@%m"
#    prompt_segment magenta black "%(!.%{%F{yellow}%}.)%n"
    prompt_segment black default ' [%n] [%*] '
  fi
}

# k8s prompt (kubens)
prompt_kubens() {
  if [[ -n $(kubectx -c 2>/dev/null) ]]; then
    local kube_ns=$(kubens -c)
    prompt_segment cyan black "kubens) ${kube_ns}"
  fi
}

# k8s prompt (kubectx)
prompt_kubectx() {
  if [[ -n $(kubectx -c 2>/dev/null) ]]; then
    local kube_ctx=$(kubectx -c)
    prompt_segment magenta black "kubectx) ${kube_ctx}"
  fi
}


# newline prompt
prompt_newline() {
  if [[ -n $CURRENT_BG ]]; then
    echo -n "%{%k%F{$CURRENT_BG}%}$SEGMENT_SEPARATOR
%{%k%F{blue}%}$SEGMENT_SEPARATOR"
  else
    echo -n "%{%k%}"
  fi

  echo -n "%{%f%}"
  CURRENT_BG=''
}

## Main prompt
build_prompt() {
  RETVAL=$?
  prompt_status
  prompt_virtualenv
  prompt_aws
  prompt_kubens
  prompt_kubectx
# prompt_context
  prompt_dir
  prompt_git
  prompt_bzr
  prompt_hg
  prompt_context
  prompt_newline
  prompt_end
}

{% endraw %} # jekyll template tag
```



## TroubleShooting

### vscode에서 폰트 깨짐 현상

https://gonigoni.kr/posts/vscode-oh-my-zsh/

### WSL2 시간 동기화 문제

https://github.com/microsoft/WSL/issues/4149

```
sudo apt install ntpdate
sudo ntpdate -sb time.nist.gov
```

