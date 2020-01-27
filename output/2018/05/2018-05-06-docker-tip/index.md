# Switch between Linux and Windows Container on D4W


If you are using Docker for Windows there might be a need to switch between Linux OR Windows container at times.


The first way to achieve this is to right click on the whale icon from the system tray and choose either the option "Switch to Linux Containers..." OR "Switch to Windows Containers..."

 ![SystemTray](/images/systemtray.png)

 The other way to achieve this from the CMD/Powershell is to execute the below command:
{{% admonition note %}}
        & 'C:\Program Files\Docker\Docker\DockerCli.exe' -SwitchDaemon
{{% /admonition %}}

![PowershellCommand](/images/powershell1.png)


{{% admonition tip %}}
There is a running process "com.docker.proxy.exe" which listens on the default named pipe : ***\\.\\pipe\\docker_engine*** [for linux containers].
{{% /admonition %}}

{{% admonition tip %}}
If you switch from Linux to Windows container the Windows Docker engine "dockerd.exe" will be started for you which listens on ***\\.\\pipe\\docker_engine_windows*** & the above proxy process redirects to this.
{{% /admonition %}}