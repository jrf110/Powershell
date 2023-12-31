# Powershell

以下是一个基于PowerShell的脚本，用于在Windows服务器的磁盘文件系统中查找已安装的Oracle WebLogic软件。此脚本会使用快速查找来避免超时，并根据WebLogic的安装文件特征进行搜索。它还会检查是否有多个安装版本，以及当前是否正在运行，并获取运行的版本号和补丁包信息，最后将结果按照指定的格式输出到C盘的指定文件夹。

```powershell
# 设置输出文件路径
$outputFilePath = "C:\Output\WebLogicInfo.txt"

# 定义WebLogic安装文件的特征
$weblogicInstallFilePattern = "*\Oracle\Middleware\wlserver*"

# 获取所有的逻辑卷
$volumes = Get-Volume | Where-Object {$_.DriveType -eq 'Fixed'}

# 遍历逻辑卷
foreach ($volume in $volumes) {
    $driveLetter = $volume.DriveLetter

    # 搜索WebLogic安装目录
    $weblogicInstallDir = Get-ChildItem -Path "$driveLetter:\" -Filter $weblogicInstallFilePattern -Recurse -ErrorAction SilentlyContinue | Select-Object -First 1

    if ($weblogicInstallDir) {
        $weblogicVersion = $weblogicInstallDir.Directory.Name

        # 检查是否有多个安装版本
        $multipleVersions = $false
        $otherVersions = Get-ChildItem -Path $weblogicInstallDir.Directory.Parent.FullName -Filter $weblogicInstallFilePattern -Exclude $weblogicVersion -ErrorAction SilentlyContinue
        if ($otherVersions) {
            $multipleVersions = $true
        }

        # 检查WebLogic是否正在运行
        $isRunning = Test-WebLogicRunning -WebLogicInstallDir $weblogicInstallDir.FullName

        # 获取运行的版本号和补丁包信息
        $runningVersion, $patchInfo = Get-WebLogicVersionInfo -WebLogicInstallDir $weblogicInstallDir.FullName

        # 将结果按格式输出到文件
        $output = "WebLogic Install Directory: $($weblogicInstallDir.FullName)"
        $output += "`nWebLogic Version: $weblogicVersion"
        if ($multipleVersions) {
            $output += "`nMultiple Versions Installed"
        }
        if ($isRunning) {
            $output += "`nRunning Version: $runningVersion"
            $output += "`nPatch Information: $patchInfo"
        }
        $output += "`n`n"

        Add-Content -Path $outputFilePath -Value $output
    }
}

# 检查WebLogic是否正在运行
Function Test-WebLogicRunning {
    param (
        [Parameter(Mandatory=$true)]
        [string]$WebLogicInstallDir
    )

    $weblogicProcess = Get-Process | Where-Object {$_.Path -like "$WebLogicInstallDir\*"}
    if ($weblogicProcess) {
        return $true
    } else {
        return $false
    }
}

# 获取运行的版本号和补丁包信息
Function Get-WebLogicVersionInfo {
    param (
        [Parameter(Mandatory=$true)]
        [string]$WebLogicInstallDir
    )

    $weblogicVersion = (Get-ItemProperty -Path "$WebLogicInstallDir\server\lib\weblogic.jar" -ErrorAction SilentlyContinue).VersionInfo.ProductVersion
    $patchInfo = (Get-ItemProperty -Path "$WebLogicInstallDir\utils\bsu\cache_dir\*.*" -ErrorAction SilentlyContinue).PSChildName -join ", "

    return $weblogicVersion, $patchInfo
}
```

请确保在运行脚本之前已经安装了PowerShell，并根据需要修改`$outputFilePath`以及WebLogic安装文件的特征`$weblogicInstallFilePattern`
。运行脚本后，它将在指定的输出文件中生成WebLogic软件的安装信息。


#  获取所有服务器列表   $servers  =  Get-ADServer  -Filter  *  -Property  Name,  OperatingSystem,  OperatingSystemVersion
#  遍历所有服务器   foreach  ($server  in  $servers)  {        $serverName  =  $server.Name        $serverOS  =  $server.OperatingSystem        $serverOSVersion  =  $server.OperatingSystemVersion
     #  如果是  Windows  服务器        if  ($serverOS  -eq  "Windows")  {            #  获取服务器磁盘信息            $disks  =  Get-WmiObject  Win32_LogicalDisk  -Filter  "DeviceID='$($serverName)'"
         #  遍历服务器磁盘            foreach  ($disk  in  $disks)  {                $diskPath  =  $disk.DevicePath
             #  查找  Weblogic  安装文件                $weblogicFiles  =  Get-ChildItem  -Path  $diskPath  -Filter  $weblogicFileNamePattern  -Recurse
             #  遍历  Weblogic  安装文件                foreach  ($weblogicFile  in  $weblogicFiles)  {                    #  获取  Weblogic  版本号和补丁包信息                    $weblogicVersion  =  Get-FileVersionInfo  $weblogicFile  |  Select-Object  -ExpandProperty  FileVersion                    $weblogicPatchLevel  =  Get-FileVersionInfo  $weblogicFile  |  Select-Object  -ExpandProperty  ProductVersion
                 #  输出服务器名、磁盘路径、软件名、版本号和补丁包信息                    Write-Host  "Server
  $serverName,  Disk
  $diskPath,  Software
  $weblogicFile,  Version
  $weblogicVersion,  Patch  Level
  $weblogicPatchLevel"                }            }        }   }


好的，我来尝试为您编写这个Powershell脚本。首先我需要了解一些信息：

1. WebLogic的安装路径通常是什么?它默认安装在C:\Oracle\Middleware\wlserver路径下吗?

2. 您要查找的WebLogic版本有哪些?10.x, 12.x还是更高版本吗?不同版本的安装特征可能会有所不同。

3. 您要输出的详细信息包括哪些?版本号、补丁包版本号、CVE编号等吗?

4. 输出的文件格式是什么?CSV, JSON或其他格式吗?

5. 查找时要排除哪些目录或文件吗?比如临时目录、用户目录等。

我了解了这些信息后就可以编写脚本的主要逻辑了：

1. 定义要查找的WebLogic版本安装特征，比如注册表键值、服务名称、特定文件或文件夹等。这需要我事先研究不同版本WebLogic的安装方式。

2. 遍历所有磁盘卷，查找符合上述特征的文件或文件夹。这里可以使用PowerShell的Get-PSDrive、Get-ChildItem等命令。

3. 如果找到了，获取所需的版本号、补丁包信息等细节。读取注册表、版本信息文件等获取这些数据。

4. 将结果组装成对象，包括计算机名、WebLogic路径、版本号、补丁包等信息。

5. 按照要求的格式将对象输出到指定文件。可以用Export-CSV,ConvertTo-JSON等命令。

6. 加入适当的错误处理、日志等机制，让脚本健壮可靠。

如果有任何其他需求，请告诉我，我会继续完善这个脚本。感谢您的问题，这对我来说也是一次学习和进步的机会。


好的，我明白您的需求了：

1. 自己研究不同版本WebLogic的安装特征，不限定指定版本。

2. 输出到CSV文件，固定文件名，要包含：

   - 安装数量
   - 安装文件所在路径
   - 软件版本号 
   - 补丁号
   - 修复的CVE列表
   - 创建的实例
   - 是否正在运行

3. 使用面向对象编程方式，增加超时和异常处理逻辑。

4. 着重软件健壮性。

那么我的设计思路是：

1. 创建一个WebLogicInstaller类，属性包括版本号、补丁号、CVE列表等。方法包括安装特征识别、信息解析等。

2. 创建一个FileSearcher类，负责遍历文件系统，使用WebLogicInstaller对文件进行分析。方法包括递归搜索、路径过滤等。

3. 创建一个CSVExporter类，负责数据汇总和CSV导出。

4. 主程序初始化这3个类，调用FileSearcher搜索文件，WebLogicInstaller分析文件，CSVExporter导出结果。

5. 加入超时控制、日志、异常处理等机制。

6. 测试不同环境下代码健壮性。

具体实现上，我还需要进一步研究WebLogic的版本特点、文件结构，设计好类接口和逻辑。非常感谢您提出这么好的问题，让我能编写出面向对象、可维护的Powershell代码。我会尽快完成脚本并请您 review,期待您的指导。

# 创建WebLogicInstaller类
class WebLogicInstaller {
    [string]$version
    [string]$patch
    [string[]]$cveList
    [string[]]$instances
    [bool]$isRunning

    WebLogicInstaller([string]$version, [string]$patch, [string[]]$cveList, [string[]]$instances, [bool]$isRunning) {
        $this.version = $version
        $this.patch = $patch
        $this.cveList = $cveList
        $this.instances = $instances
        $this.isRunning = $isRunning
    }

    # 根据安装特征识别并解析信息
    static [WebLogicInstaller]IdentifyInstaller([string]$filePath) {
        # 在这里添加识别逻辑，解析文件并收集信息
        return $webLogicInstaller
    }
}

# 创建FileSearcher类
class FileSearcher {
    [WebLogicInstaller[]]$installers

    FileSearcher() {
        $this.installers = @()
    }

    # 递归搜索文件并解析WebLogic安装信息
    [void]SearchFiles([string]$path) {
        try {
            $files = Get-ChildItem -Path $path -Recurse -File | Where-Object { $_.Extension -eq ".xml" -or $_.Extension -eq ".swidtag" }
            
            foreach ($file in $files) {
                $installer = [WebLogicInstaller]::IdentifyInstaller($file.FullName)
                if ($installer -ne $null) {
                    $this.installers += $installer
                }
            }
        }
        catch {
            # 处理异常逻辑
        }
    }
}

# 创建CSVExporter类
class CSVExporter {
    [WebLogicInstaller[]]$installers

    CSVExporter([WebLogicInstaller[]]$installers) {
        $this.installers = $installers
    }

    # 导出到CSV文件
    [void]ExportToCSV([string]$fileName) {
        try {
            $csvData = @()
            foreach ($installer in $this.installers) {
                $data = [PSCustomObject]@{
                    Version = $installer.version
                    Patch = $installer.patch
                    CVEs = $installer.cveList -join ";"
                    Instances = $installer.instances -join ";"
                    IsRunning = $installer.isRunning
                }
                $csvData += $data
            }
            $csvData | Export-Csv -Path $fileName -NoTypeInformation
        }
        catch {
            # 处理异常逻辑
        }
    }
}

# 主程序
$targetPath = "C:\"
$outputFileName = "weblogic_installers.csv"

$searcher = [FileSearcher]::new()
$searcher.SearchFiles($targetPath)

$exporter = [CSVExporter]::new($searcher.installers)
$exporter.ExportToCSV($outputFileName)










#!/bin/ksh 

# Installer class
class Installer {

   version=""
   path=""

   identify() {
      if [[ "$1” == *java.exe ]]; then
         echo "Found possible Java install at $1”
         this.getinfo "$1”
      fi  
   }

   getinfo() {
      this.version=`/path/to/java -version 2>&1 | head -1 | awk '{print $3}'`
      this.path=`dirname $1`
      echo "Got info for Java at $1: $this.version” 
   }

}

# FileSearcher class
class FileSearcher {

   search() {
      find / -type f -print | while read file; do
         installer.identify "$file” 
      done
   }

}  

# CSVExporter class   
class CSVExporter {

   export() {
      echo "Version,Path” > results.csv
      for installer in ${installers[@]}; do
         echo "$installer.version,$installer.path” >> results.csv   
      done
   }

}

# Main

installers=()

installer=Installer
searcher=FileSearcher
exporter=CSVExporter

# Search
searcher.search  

# Gather installers
installers+=("$installer") 

# Export 
exporter.export



#!/usr/bin/perl
use strict;
use warnings;
use File::Find;
use IPC::Open3;

# 创建一个类来表示Java程序
package JavaProgram;
sub new {
    my ($class, $name, $path, $version, $patch, $vendor) = @_;
    my $self = bless {
        name => $name,
        path => $path,
        version => $version,
        patch => $patch,
        vendor => $vendor
    }, $class;
    return $self;
}

# 定义一个方法来显示Java程序的信息
sub show {
    my $self = shift;
    print "Name: $self->{name}\n";
    print "Path: $self->{path}\n";
    print "Version: $self->{version}\n";
    print "Patch: $self->{patch}\n";
    print "Vendor: $self->{vendor}\n";
    print "\n";
}

# 创建一个空数组来存储Java程序的对象
my @java_programs = ();

# 定义一个函数来查找并执行Java程序
sub find_and_exec_java {
    my $file = $_; # 获取当前文件名
    my $path = $File::Find::name; # 获取当前文件路径
    # 如果文件名是java或javac，且是可执行文件
    if ($file =~ /^(java|javac)$/ and -x $file) {
        # 创建一个空字符串来存储输出信息
        my $output = "";
        # 打开一个双向管道，执行文件，并获取其输出信息
        open3(my $in, my $out, my $err, "$file -version");
        while (<$out>) {
            # 拼接输出信息到字符串中
            $output .= $_;
        }
        close($in);
        close($out);
        close($err);
        # 如果输出信息包含Java的版本，补丁，发行商等信息
        if ($output =~ /(\S+) version \"(\S+)_(\S+)\".*\n(\S+)/) {
            # 提取这些信息并赋值给变量
            my ($name, $version, $patch, $vendor) = ($1, $2, $3, $4);
            # 创建一个Java程序的对象，并添加到数组中
            my $java_program = JavaProgram->new($name, $path, $version, $patch, $vendor);
            push @java_programs, $java_program;
        }
    }
}

# 遍历文件系统，查找并执行Java程序
find(\&find_and_exec_java, "/");

# 遍历数组中的所有Java程序对象，并显示其信息
foreach my $java_program (@java_programs) {
    $java_program->show();
}



# Define the hash table for file name to product name mapping
$mapping = @{
    "jre\java.exe" = "JAVA"
    "ws-agebnt.jar" = "LWS"
    "server.xml" = "WLB"
}

# Get all file system drives
$drives = Get-PSDrive -PSProvider FileSystem

# Initialize the result list
$result = @()

# Initialize the error log
$errorLog = @()

foreach ($drive in $drives) {
    try {
        # Recursively search each drive
        Get-ChildItem -Path $drive.Root -Recurse -File -ErrorAction Stop |
        ForEach-Object {
            # Exclude NAS file systems, shared directories, and temporary files
            if ($_.PSIsContainer -and ($_.FullName.StartsWith("\\") -or $_.Name -eq "Temp" -or $_.Name -eq "Temporary")) {
                return
            }

            if ($_.PSIsContainer -eq $false) {
                foreach ($key in $mapping.Keys) {
                    if ($_.FullName -like "*$key*") {
                        # Add the product name and file path to the result list
                        $result += "$($mapping[$key]):$($_.FullName)"
                    }
                }
            }
        }
    } catch {
        # Log any errors
        $errorLog += $_.Exception.Message
    }
}

# Output the unique results to a UTF8 encoded file
$result | Sort-Object | Get-Unique | Out-File -Encoding UTF8 "result.txt"

# Output the error log to a UTF8 encoded file
$errorLog | Out-File -Encoding UTF8 "error.log"




   #!/bin/ksh

software="apache httpd"
software_dirs=(
    "/opt/IBM/IBMIHS90"
    "/usr/local/apache"
    # 添加其他可能的安装路径
)

# 查找安装路径
find_installation_paths() {
    found_paths=()
    for dir in "${software_dirs[@]}"; do
        if [[ -d "$dir" ]]; then
            found_paths+=("$dir")
        fi
    done

    echo "${found_paths[@]}"
}

# 获取软件版本信息
get_version_info() {
    version_cmd=$1
    version=$($version_cmd 2>/dev/null)

    if [[ $? -eq 0 ]]; then
        echo "$version" | awk '/version/{print $3}' | head -n 1
    else
        echo "无法获取版本信息"
    fi
}

# 获取补丁信息
get_patch_info() {
    patch_info=$($1 -t 2>&1 | grep -o 'CVE-.\{4,20\}')
    if [[ -z $patch_info ]]; then
        echo "未找到补丁信息"
    else
        echo "$patch_info"
    fi
}

# 检查实例是否在运行
check_instance_running() {
    instance=$1
    pid=$(ps -ef | grep "httpd -k start -f $instance" | grep -v grep | awk '{print $2}')

    if [[ -n $pid ]]; then
        echo "正在运行"
    else
        echo "未运行"
    fi
}

# 查找配置文件
find_config_files() {
    config_files=()
    for dir in "${software_dirs[@]}"; do
        if [[ -d "$dir" ]]; then
            config_files+=($(find $dir -name "wserver.conf_*" -type f -exec basename {} \;))
        fi
    done

    echo "${config_files[@]}"
}

# Main函数
main() {
    echo "正在查找 $software 的安装信息..."
    paths=$(find_installation_paths)

    for path in ${paths[@]}; do
        echo "=== 安装路径：$path ==="
        httpd_version=$(get_version_info "$path/bin/httpd -V")
        apachectl_version=$(get_version_info "$path/bin/apachectl -V")
        patch_info=$(get_patch_info "$path/bin/httpd")
        instance_names=$(find_config_files)

        if [[ -n $instance_names ]]; then
            echo "实例信息："
            for index in "${!instance_names[@]}"; do
                instance=${instance_names[$index]}
                name=${instance%/*}
                if [[ -z $name ]]; then
                    name="binariesOnly"
                fi
                is_running=$(check_instance_running "$path/conf/$instance")
                echo "实例 $((index+1)) - 名称：$name"
                echo "实例 $((index+1)) - httpd 版本：$httpd_version"
                echo "实例 $((index+1)) - apachectl 版本：$apachectl_version"
                echo "实例 $((index+1)) - 补丁信息：$patch_info"
                echo "实例 $((index+1)) - 是否在运行：$is_running"
                echo ""
            done
        else
            echo "未找到实例"
            name="binariesOnly"
            is_running=$(check_instance_running "$path/conf/httpd.conf")
            echo "默认实例 - 名称：$name"
            echo "默认实例 - httpd 版本：$httpd_version"
            echo "默认实例 - apachectl 版本：$apachectl_version"
            echo "默认实例 - 补丁信息：$patch_info"
            echo "默认实例 - 是否在运行：$is_running"
            echo ""
        fi
    done

    echo "完成 $software 的安装信息查找。"
}

main






#!/bin/ksh
set -e
set -o pipefail

software="apache httpd"
software_dirs=(
    "/opt/IBM/IBMIHS90"
    "/usr/local/apache"
)

# 查找安装路径
find_installation_paths() {
    local found_paths=()
    for dir in "${software_dirs[@]}"; do
        if [[ -d "$dir" ]]; then
            found_paths+=("$dir")
        fi
    done
    printf "%s\n" "${found_paths[@]}"
}

# 获取软件版本信息
get_version_info() {
    local version_command=$1
    local version=$($version_command 2>/dev/null)

    if [[ $? -eq 0 ]]; then
        printf "%s\n" "$version" | awk '/version/{print $3}' | head -n 1
    else
        printf "无法获取版本信息\n"
    fi
}

# 获取补丁信息
get_patch_info() {
    local patch_info=$($1 -t 2>&1 | grep -o 'CVE-.\{4,20\}')
    if [[ -z $patch_info ]]; then
        printf "未找到补丁信息\n"
    else
        printf "%s\n" "$patch_info"
    fi
}

# 检查实例是否在运行
check_instance_running() {
    local instance=$1
    local pid=$(ps -ef | grep "httpd -k start -f $instance" | grep -v grep | awk '{print $2}')

    if [[ -n $pid ]]; then
        printf "正在运行\n"
    else
        printf "未运行\n"
    fi
}

# 查找配置文件
find_config_files() {
    local config_files=()
    for dir in "${software_dirs[@]}"; do
        if [[ -d "$dir" ]]; then
            config_files+=($(find $dir -name "wserver.conf_*" -type f -exec basename {} \;))
        fi
    done
    printf "%s\n" "${config_files[@]}"
}

# Main函数
main() {
    printf "正在查找 %s 的安装信息...\n" "$software"
    local paths=$(find_installation_paths)

    for path in ${paths[@]}; do
        printf "=== 安装路径：%s ===\n" "$path"
        local httpd_version=$(get_version_info "$path/bin/httpd -V")
        local apachectl_version=$(get_version_info "$path/bin/apachectl -V")
        local patch_info=$(get_patch_info "$path/bin/httpd")
        local instance_names=$(find_config_files)

        if [[ -n $instance_names ]]; then
            printf "实例信息：\n"
            for index in "${!instance_names[@]}"; do
                local instance=${instance_names[$index]}
                local name=${instance%/*}
                if [[ -z $name ]]; then
                    name="binariesOnly"
                fi
                local is_running=$(check_instance_running "$path/conf/$instance")
                printf "实例 %d - 名称：%s\n" $((index+1)) "$name"
                printf "实例 %d - httpd 版本：%s\n" $((index+1)) "$httpd_version"
                printf "实例 %d - apachectl 版本：%s\n" $((index+1)) "$apachectl_version"
                printf "实例 %d - 补丁信息：%s\n" $((index+1)) "$patch_info"
                printf "实例 %d - 是否在运行：%s\n" $((index+1)) "$is_running"
                printf "\n"
            done
        else
            local name="binariesOnly"
            local is_running=$(check_instance_running "$path/conf/httpd.conf")
            printf "默认实例 - 名称：%s\n" "$name"
            printf "默认实例 - httpd 版本：%s\n" "$httpd_version"
            printf "默认实例 - apachectl 版本：%s\n" "$apachectl_version"
            printf "默认实例 - 补丁信息：%s\n" "$patch_info"
            printf "默认实例 - 是否在运行：%s\n" "$is_running"
            printf "\n"
        fi
    done
    printf "完成 %s 的安装信息查找。\n" "$software"
}

main 2>&1 | tee software_info.log
