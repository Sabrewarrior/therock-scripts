# Building ROCm/TheRock on Windows Server 2022 Standard

All commands are for cmd not powershell

Download Chocolatey and turn on Long Paths
```
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "[System.Net.ServicePointManager]::SecurityProtocol = 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
powershell -noprofile -nologo -c New-ItemProperty -Path "HKLM:\\SYSTEM\\CurrentControlSet\\Control\\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
```

Install required tools using Chocolatey
```
choco install visualstudio2022buildtools -y --params "--add Microsoft.VisualStudio.Component.VC.Tools.x86.x64 --add Microsoft.VisualStudio.Component.VC.CMake.Project --add Microsoft.VisualStudio.Component.VC.ATL --add Microsoft.VisualStudio.Component.Windows11SDK.22621"
choco install git.install -y --params "'/GitAndUnixToolsOnPath /Symlinks'"
git config --global core.longpaths true
choco install cmake --version=3.31.0 -y
refreshenv
choco install ninja -y
choco install ccache -y
choco install sccache -y
choco install python -y
choco install strawberryperl -y
refreshenv
```

Clone PAL and TheRock. Download the python script from this repo
```
cd /
mkdir work
cd work
git clone https://github.com/nod-ai/amdgpu-windows-interop.git
git clone https://github.com/ROCm/TheRock.git
cd TheRock
curl -O -L https://raw.githubusercontent.com/Sabrewarrior/therock-scripts/refs/heads/main/therock-build.py
```

Create python venv and run fetch sources for subprojects and applying patches
```
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
python ./build_tools/fetch_sources.py
```

Load Visual Studio environment variables and build using the python script
```
"%ProgramFiles(x86)%\Microsoft Visual Studio\2022\BuildTools\Common7\Tools\VsDevCmd.bat" -arch=amd64
python therock-build.py -DTHEROCK_AMDGPU_FAMILIES=gfx1100 -DCMAKE_MSVC_DEBUG_INFORMATION_FORMAT=Embedded
```
