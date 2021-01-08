# AutoRclone：使用服务帐户进行rclone复制/移动/同步（自动）（仍处于beta阶段）
非常感谢[rclone]（https://rclone.org/）和[folderclone]（https://github.com/Spazzlo/folderclone）。

-[x]使用脚本创建服务帐户
-[x]将大量服务帐户添加到rclone配置文件中
-[x]将大量服务帐户添加到贵组织的组中
-[x]在rclone复制/移动/同步时自动切换帐户 
-[x] Windows系统受支持

步骤1.将代码复制到您的VPS或本地计算机
---------------------------------
_首先，请先安装python3。因为我们使用python作为编程语言。_

**对于Linux系统**：安装
[屏幕]（https://www.interserver.net/tips/kb/using-screen-to-attach-and-detach-console-sessions/），`git`和 [最新的rclone]（https://rclone.org/downloads/#script-download-and-install）。 
如果在Debian / Ubuntu中，请直接使用此命令
```
sudo apt-get install screen git && curl https://rclone.org/install.sh | sudo bash
```
成功安装以上所有依赖项后，运行此命令
```
sudo git clone https://github.com/cgkings/AutoRclone && cd AutoRclone && sudo pip3 install -r requirements.txt
```
**对于Windows系统**：直接下载此项目，然后[安装最新的rclone]（https://rclone.org/downloads/）。 
然后在我们的项目文件夹中运行此命令（在cmd命令窗口或PowerShell窗口中键入）
```
pip3 install -r requirements.txt
```

步骤2.生成服务帐户[什么是服务帐户]（https://cloud.google.com/iam/docs/service-accounts）[如何在rclone中使用服务帐户]（https://rclone.org/drive/#service-account-support）。
---------------------------------
让我们仅创建所需的服务帐户。 
**警告：**滥用此功能不是autorclone的目的，我们不**建议您进行大量的项目，仅一个项目和100 sa就可以使您大量使用，这也可能导致滥用可能会使您的项目被Google禁止。 


在[Python快速入门]（https://developers.google.com/drive/api/v3/quickstart/python）中启用Drive API
并将文件“ credentials.json”保存到项目目录中。

如果您的帐户中没有任何项目，则 
*创建1个项目
*启用所需的服务
*创建100个（1个项目，每个项目有100个）服务帐户
*将credentials文件下载到名为 `accounts`的文件夹中

```
注意：1个服务帐户每天可以复制大约750GB，1个项目可以创建100个服务帐户，因此每天需要复制75TB，对于大多数用户来说，这应该很容易满足。 
```

该命令看起来像 
 `python3 gen_sa_accounts.py --quick-setup 1`
 将“ 1”替换为所需的项目数

如果您已经有N个项目，并且只想在新创建的项目中创建服务帐户，
那么
*创建其他1个项目（项目N + 1到项目N + 2）
*启用所需的服务
*创建100个（1个项目，有100个）服务帐户
*并将credentials文件下载到名为 `accounts`的文件夹中

run 

`python3 gen_sa_accounts.py --quick-setup 1 --new-only` 
如果要使用现有项目创建一些服务帐户（不要创建更多项目），请运行 
python3 gen_sa_accounts.py --quick-setup -1
请注意，这将覆盖现有的服务帐户。

完成后，在一个名为`accounts`的文件夹中将有许多json文件。 


第3步。将服务帐户添加到Google网上论坛（可选，但建议长期无忧使用）
---------------------------------
考虑到以下原因，我们使用Google网上论坛来管理我们的服务帐户：  
[Team Drive成员的正式限制]（https://support.google.com/a/answer/7338880?hl=zh_CN）（直接添加为成员的个人和小组的限制：600）。

####对于GSuite管理员
1.按照[官方步骤]（https://developers.google.com/admin-sdk/directory/v1/quickstart/python）打开Directory API（将生成的json文件保存到“ credentials”文件夹中）。

2. [在管理控制台中]为您的组织创建组（https://support.google.com/a/answer/33343?hl=zh_CN）。创建组后，您将拥有一个地址，例如`sa @ yourdomain.com`。

3.运行`python3 add_to_google_group.py -g sa @ yourdomain.com`

_对于上述标志的含义，请运行`python3 add_to_google_group.py -h`_

####对于普通用户

创建[Google网上论坛]（https://groups.google.com/），然后手动将服务帐户添加为成员。
每天一次限制为10个，每天限制为100个，但是如果您阅读上述警告和注意事项，则将有1个项目，因此很容易就在您的范围内。 

步骤4.将服务帐户或Google网上论坛添加到Team Drive中
---------------------------------
_如果您不使用Team Drive，请跳过。
**警告：**不**不建议使用服务帐户将“复制到”团队驱动器中的文件夹克隆到，SA最适合团队驱动器。 

如果您已经创建了Google网上论坛（**第2步**）来管理您的服务帐户，请将网上论坛地址“ sa@yourdomain.com”或“ sa@googlegroups.com”添加到源团队驱动器（tdsrc）和目标地址团队驱动力（tddst）。 
 
否则，将服务帐户直接添加到Team Drive中。
>在[Python快速入门]（https://developers.google.com/drive/api/v3/quickstart/python）中启用Drive API 
并在步骤2 **中将`credentials.json`保存到项目根路径中。
>-将服务帐户添加到源Team Drive中：
`python3 add_to_team_drive.py -d SharedTeamDriveSrcID`
>-将服务帐户添加到目标Team Drive：
`python3 add_to_team_drive.py -d SharedTeamDriveDstID`

步骤5.开始您的任务
---------------------------------
让我们使用服务帐户复制数百个TB资源。 
**注意**：讽刺，过度使用它（不管您使用什么克隆脚本）可能会引起Google的注意，我们建议您不要贪吃，克隆重要的东西，而不要下载整个Wikipedia。

####用于服务器端复制
-[x]公开共享的文件夹到Team Drive
-[x]团队驱动到团队驱动
-[]公共共享文件夹到公共共享文件夹（具有写权限）
-[] Team Drive到公开共享的文件夹
```
python3 rclone_sa_magic.py -s SourceID -d DestinationID -dp DestinationPathName -b 1 -e 600
```
-_对于上述标志的含义，请运行python3 rclone_sa_magic.py -h_

-_添加`--disable_list_r`如果`rclone` [无法读取公共共享文件夹的所有内容]（https://forum.rclone.org/t/rclone-cannot-see-all-files-folder-in-public-共享文件夹/12351）。_

-_请确保Rclone可以读取您的源目录和目标目录。使用`rclone size`：_检查它

1.```rclone --config rclone.conf size --disable ListR src001：```

2.```rclone --config rclone.conf size --disable ListR dst001：```

####对于Google云端硬盘本地（需要进行一些测试）
-[x]在Team Drive本地
-[]在私有文件夹本地
-[]专用文件夹（任何服务帐户都不能对专用文件夹执行任何操作）
```
python3 rclone_sa_magic.py -sp YourLocalPath -d DestinationID -dp DestinationPathName -b 1 -e 600
```

*运行命令“ tail -f log_rclone.txt”以查看详细信息（仅Linux）。

！[]（AutoRclone.jpg）

还让我们在Telegram Group [AutoRclone]（https://t.me/AutoRclone）中谈论这个项目

[博客（中文）]（博客（中文） 
https://gsuitems.com/index.php/archives/13/）| [Google云端硬盘群组]（https://t.me/google_drive）| [Google云端硬盘频道]（https://t.me/gdurl）  



