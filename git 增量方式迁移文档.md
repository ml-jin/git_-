# git 增量方式上传

[TOC]

**增量迁移存储库**

在迁移到 AWS CodeCommit 时，请考虑以增量或数据块方式推送存储库，以减少因间歇性网络问题或网络性能下降而导致整个推送操作失败的几率。通过使用类似此处包含的脚本进行增量推送，您可以重新启动迁移，并只推送先前推送失败的提交。

本主题中的过程向您展示如何创建和运行以增量方式迁移存储库的脚本，该脚本将只重新推送那些推送失败的增量提交，直到迁移完成。

编写这些说明时，假定您已完成[设置 ](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/setting-up.html)和[创建  存储库](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-create-repository.html)中的步骤。                                  

**主题**

- [步骤0: 确定是否逐步迁移](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-push-large-repositories.html#how-to-push-large-repositories-determine)
- [第1步: 安装先决条件并添加 CodeCommit 存储库作为远端 ](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-push-large-repositories.html#how-to-push-large-repositories-prereq)
- [第2步: 创建用于增量迁移的脚本](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-push-large-repositories.html#how-to-push-large-repositories-createscript)
- [第3步: 运行脚本并逐步迁移到 CodeCommit](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-push-large-repositories.html#how-to-push-large-repositories-runscript)
- [附件: 脚本示例 incremental-repo-migration.py ](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-push-large-repositories.html#how-to-push-large-repositories-sample)

## 步骤0: 确定是否逐步迁移

要确定存储库的整体大小以及是否需要增量迁移，可以考虑以下几个因素。首先要考虑的当然是存储库中项目的整体大小。存储库的累积历史记录等因素也会对大小产生影响。就算存储库的各个资产并不大，但如果它包含多年的历史记录和分支，其整体大小也可能非常大。您可以通过多种策略来更简单、更高效地迁移这些存储库。例如，可以在克隆具有长期开发历史的存储库时使用浅克隆策略，也可以关闭大型二进制文件的增量压缩。您可以通过查阅                                     Git 文档来研究选项，也可以选择使用本主题中包含的示例脚本 (`incremental-repo-migration.py`) 来设置和配置以增量推送方式迁移存储库。                                  

如果您满足以下一个或多个条件，则可能需要配置增量推送：

- 您要迁移的存储库具有五年以上的历史。
- 您的 Internet 连接存在间歇性中断、丢包、响应缓慢或其他服务中断问题。
- 存储库的整体大小大于 2 GB，并且您打算迁移整个存储库。
- 存储库包含压缩率不高的大型项目或二进制文件，例如具有超过五个跟踪版本的大型映像文件。
- 您以前尝试过迁移到 CodeCommit，但收到“内部服务错误”消息。

就算上述条件都不成立，您也可以选择增量推送。

## 第1步: 安装先决条件并添加 CodeCommit 存储库作为远端 

您可以创建自己的自定义脚本，它有自己的必备组件。如果您使用本主题中包括的示例，则必须：

- 安装其必备组件。
- 将存储库克隆到本地计算机。
- 将 CodeCommit 存储库添加为要迁移的存储库的远程存储库。

**设置运行 incremental-repo-migration.py**

1.  在本地计算机上，安装 Python 2.6 或更高版本。有关更多信息以及最新版本的信息，请参阅 [Python 网站](https://www.python.org/downloads/)

。                                           

在同一台计算机上,安装 GitPython,这是一个用于与Git存储库交互的Python库。有关更多信息，请参阅 [GitPython 文档。](http://gitpython.readthedocs.org/en/stable/)

 使用 **git clone --mirror** 命令克隆要迁移到本地计算机的存储库。在终端（Linux, macOS, or Unix）或命令提示符 (Windows) 中，使用 **git clone --mirror** 命令为该存储库创建一个本地存储库，包括要在其中创建本地存储库的目录。例如,复制名为的Git存储库 `MyMigrationRepo` URL为 `https://example.com/my-repo/` 到名为的目录 `my-repo`:                                           

```

git clone --mirror https://example.com/my-repo/MyMigrationRepo.git my-repo
```

您应会看到类似以下内容的输出，这表示存储库已被克隆到名为 my-repo 的本地空存储库中：

```

Cloning into bare repository 'my-repo'...
remote: Counting objects: 20, done.
remote: Compressing objects: 100% (17/17), done.
remote: Total 20 (delta 5), reused 15 (delta 3)
Unpacking objects: 100% (20/20), done.
Checking connectivity... done.        
```

将目录更改为刚克隆的存储库的本地存储库(例如,`my-repo`)。从该目录中,使用 **git remote add DefaultRemoteName                                                 RemoteRepositoryURL** 命令添加 CodeCommit 作为本地存储库的远程存储库。                                           

​                                                 

注意

在推送大型存储库时，请考虑使用 SSH 而不是 HTTPS。在推送较大的更改、大量更改或大型存储库时，长时间运行的 HTTPS 连接通常会因为网络问题或防火墙设置而提前终止。有关为                                                    CodeCommit 设置 SSH 连接的更多信息，请参阅[对于 Linux, macOS, or Unix 上的 SSH 连接](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/setting-up-ssh-unixes.html)或[适用于 Windows 上的 SSH 连接](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/setting-up-ssh-windows.html)。                                                 

 例如，使用下面的命令来为名为 MyDestinationRepo 的 CodeCommit 存储库（它是名为 `codecommit` 的远程存储库的远程存储库）添加 SSH 终端节点：                                           

```

git remote add codecommit ssh://git-codecommit.us-east-2.amazonaws.com/v1/repos/MyDestinationRepo
```

​                                                 

提示

由于这是克隆存储库，已使用默认的远程存储库名称 (`origin`)。您必须使用其他的远程存储库名称。虽然示例使用了 `codecommit`，但您可以使用任意名称。使用 **git remote show** 命令查看为本地存储库设置的远程存储库的列表。                                                 

使用 **git remote -v** 命令显示本地存储库的提取和推送设置，确认它们设置正确。例如：                                           

```

codecommit  ssh://git-codecommit.us-east-2.amazonaws.com/v1/repos/MyDestinationRepo (fetch)
codecommit  ssh://git-codecommit.us-east-2.amazonaws.com/v1/repos/MyDestinationRepo (push)      
```

​                                                 

1. 提示

   如果您仍看到其他远程存储库的提取和推送条目（例如，origin 的条目），请使用 **git remote set-url                                                       --delete** 命令删除它们。                                                 

## 第2步: 创建用于增量迁移的脚本

编写这些步骤时，假定您使用的是 `incremental-repo-migration.py` 示例脚本。                                  

1. 打开一个文本编辑器，将[示例脚本](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-push-large-repositories.html#how-to-push-large-repositories-sample)的内容粘贴到一个空文档中。                                           
2. 将文档保存在文档目录(而非本地存储库的工作目录)中并命名 `incremental-repo-migration.py`。确保您选择的目录是在本地环境或路径变量中配置的目录,以便您可以从命令行或终端运行Python脚本。                                           

## 第3步: 运行脚本并逐步迁移到 CodeCommit

 现在，您已创建 `incremental-repo-migration.py` 脚本，可以使用它将本地存储库增量迁移到 CodeCommit 存储库。默认情况下，该脚本以 1000 个提交为批次推送提交，并尝试将运行该脚本时所在目录的 Git                                     设置用作本地存储库和远程存储库的设置。如果需要，您可以使用 `incremental-repo-migration.py` 中包含的选项配置其他设置。                                  

1. 在终端或命令提示符中，切换到要迁移的本地存储库的目录。

2. 从该目录运行以下命令：

   ```
   
   ```

1. ```
   python incremental-repo-migration.py
   ```

2. 脚本运行并在终端或命令提示符中显示进度。一些大型存储库在显示进度时会有延迟。如果单次推送失败三次，脚本将停止运行。然后，您可以重新运行脚本，它会从失败的批次继续。您可以重新运行脚本，直到所有推送成功并且迁移完成。

​                                        

提示

您可以在任意目录中运行 `incremental-repo-migration.py`，前提是您用 `-l` 和 `-r` 选项指定了要使用的本地和远程存储库设置。例如,使用来自任何目录的脚本迁移位于/tmp/`my-repo` 到远程昵称 `codecommit`:                                        

```

python incremental-repo-migration.py -l "/tmp/my-repo" -r "codecommit" 
```

 您可能还需要使用 `-b` 选项来更改增量推送时使用的默认批处理大小。例如，如果您定期推送具有经常发生更改的超大二进制文件的存储库，并且在网络带宽受限的位置执行操作，则您可能需要使用 `-b` 选项将批处理大小更改为 500 而不使用 1000。例如：                                        

```

python incremental-repo-migration.py -b 500
```

这将以 500 个提交为批次增量推送本地存储库。如果您在迁移存储库时决定再次更改批次大小（例如，在推送失败后，您决定减小批次大小），请记得使用 `-c` 选项删除批次标签，然后再使用 `-b` 重置批次大小：                                        

```

python incremental-repo-migration.py -c
python incremental-repo-migration.py -b 250
```

​                                        

重要

推送失败后重新运行脚本时，请勿使用 `-c` 选项。`-c` 选项会删除用于对提交进行批处理的标签。仅当您想要更改批处理大小并重新开始时，或者您决定不再使用该脚本时，才能使用 `-c` 选项。                                        

## 附件: 脚本示例 `incremental-repo-migration.py`                                   

为方便您参考，我们开发了一个用于增量推送存储库的示例 Python 脚本 `incremental-repo-migration.py`。该脚本是一个开源代码示例，按原样提供。                                  

```python

# Copyright 2015 Amazon.com, Inc. or its affiliates. All Rights Reserved. Licensed under the Amazon Software License (the "License"). 
# You may not use this file except in compliance with the License. A copy of the License is located at 
#    http://aws.amazon.com/asl/ 
# This file is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, express or implied. See the License for
# the specific language governing permissions and limitations under the License.

#!/usr/bin/env python

import os
import sys
from optparse import OptionParser
from git import Repo, TagReference, RemoteProgress, GitCommandError

class PushProgressPrinter(RemoteProgress):
    def update(self, op_code, cur_count, max_count=None, message=''):
        op_id = op_code & self.OP_MASK
        stage_id = op_code & self.STAGE_MASK
        if op_id == self.WRITING and stage_id == self.BEGIN:
            print("\tObjects: %d" % max_count)

class RepositoryMigration:

    MAX_COMMITS_TOLERANCE_PERCENT = 0.05
    PUSH_RETRY_LIMIT = 3
    MIGRATION_TAG_PREFIX = "codecommit_migration_"

    def migrate_repository_in_parts(self, repo_dir, remote_name, commit_batch_size, clean):
        self.next_tag_number = 0
        self.migration_tags = []
        self.walked_commits = set()
        self.local_repo = Repo(repo_dir)
        self.remote_name = remote_name
        self.max_commits_per_push = commit_batch_size
        self.max_commits_tolerance = self.max_commits_per_push * self.MAX_COMMITS_TOLERANCE_PERCENT

        try:
            self.remote_repo = self.local_repo.remote(remote_name)
            self.get_remote_migration_tags()
        except (ValueError, GitCommandError):
            print("Could not contact the remote repository. The most common reasons for this error are that the name of the remote repository is incorrect, or that you do not have permissions to interact with that remote repository.")
            sys.exit(1)

        if clean:
            self.clean_up(clean_up_remote=True)
            return

        self.clean_up()

        print("Analyzing repository")
        head_commit = self.local_repo.head.commit
        sys.setrecursionlimit(max(sys.getrecursionlimit(), head_commit.count()))

        # tag commits on default branch
        leftover_commits = self.migrate_commit(head_commit)
        self.tag_commits([commit for (commit, commit_count) in leftover_commits])

        # tag commits on each branch
        for branch in self.local_repo.heads:
            leftover_commits = self.migrate_commit(branch.commit)
            self.tag_commits([commit for (commit, commit_count) in leftover_commits])

        # push the tags
        self.push_migration_tags()

        # push all branch references
        for branch in self.local_repo.heads:
            print("Pushing branch %s" % branch.name)
            self.do_push_with_retries(ref=branch.name)

        # push all tags
        print("Pushing tags")
        self.do_push_with_retries(push_tags=True)

        self.get_remote_migration_tags()
        self.clean_up(clean_up_remote=True)

        print("Migration to CodeCommit was successful")

    def migrate_commit(self, commit):
        if commit in self.walked_commits:
            return []

        pending_ancestor_pushes = []
        commit_count = 1

        if len(commit.parents) > 1:
            # This is a merge commit
            # Ensure that all parents are pushed first
            for parent_commit in commit.parents:
                pending_ancestor_pushes.extend(self.migrate_commit(parent_commit))
        elif len(commit.parents) == 1:
            # Split linear history into individual pushes
            next_ancestor, commits_to_next_ancestor = self.find_next_ancestor_for_push(commit.parents[0])
            commit_count += commits_to_next_ancestor
            pending_ancestor_pushes.extend(self.migrate_commit(next_ancestor))

        self.walked_commits.add(commit)

        return self.stage_push(commit, commit_count, pending_ancestor_pushes)

    def find_next_ancestor_for_push(self, commit):
        commit_count = 0

        # Traverse linear history until we reach our commit limit, a merge commit, or an initial commit
        while len(commit.parents) == 1 and commit_count < self.max_commits_per_push and commit not in self.walked_commits:
            commit_count += 1
            self.walked_commits.add(commit)
            commit = commit.parents[0]

        return commit, commit_count

    def stage_push(self, commit, commit_count, pending_ancestor_pushes):
        # Determine whether we can roll up pending ancestor pushes into this push
        combined_commit_count = commit_count + sum(ancestor_commit_count for (ancestor, ancestor_commit_count) in pending_ancestor_pushes)

        if combined_commit_count < self.max_commits_per_push:
            # don't push anything, roll up all pending ancestor pushes into this pending push
            return [(commit, combined_commit_count)]

        if combined_commit_count <= (self.max_commits_per_push + self.max_commits_tolerance):
            # roll up everything into this commit and push
            self.tag_commits([commit])
            return []

        if commit_count >= self.max_commits_per_push:
            # need to push each pending ancestor and this commit
            self.tag_commits([ancestor for (ancestor, ancestor_commit_count) in pending_ancestor_pushes])
            self.tag_commits([commit])
            return []

        # push each pending ancestor, but roll up this commit
        self.tag_commits([ancestor for (ancestor, ancestor_commit_count) in pending_ancestor_pushes])
        return [(commit, commit_count)]

    def tag_commits(self, commits):
        for commit in commits:
            self.next_tag_number += 1
            tag_name = self.MIGRATION_TAG_PREFIX + str(self.next_tag_number)

            if tag_name not in self.remote_migration_tags:
                tag = self.local_repo.create_tag(tag_name, ref=commit)
                self.migration_tags.append(tag)
            elif self.remote_migration_tags[tag_name] != str(commit):
                print("Migration tags on the remote do not match the local tags. Most likely your batch size has changed since the last time you ran this script. Please run this script with the --clean option, and try again.")
                sys.exit(1)

    def push_migration_tags(self):
        print("Will attempt to push %d tags" % len(self.migration_tags))
        self.migration_tags.sort(key=lambda tag: int(tag.name.replace(self.MIGRATION_TAG_PREFIX, "")))
        for tag in self.migration_tags:
            print("Pushing tag %s (out of %d tags), commit %s" % (tag.name, self.next_tag_number, str(tag.commit)))
            self.do_push_with_retries(ref=tag.name)

    def do_push_with_retries(self, ref=None, push_tags=False):
        for i in range(0, self.PUSH_RETRY_LIMIT):
            if i == 0:
                progress_printer = PushProgressPrinter()
            else:
                progress_printer = None

            try:
                if push_tags:
                    infos = self.remote_repo.push(tags=True, progress=progress_printer)
                elif ref is not None:
                    infos = self.remote_repo.push(refspec=ref, progress=progress_printer)
                else:
                    infos = self.remote_repo.push(progress=progress_printer)

                success = True
                if len(infos) == 0:
                    success = False
                else:
                    for info in infos:
                        if info.flags & info.UP_TO_DATE or info.flags & info.NEW_TAG or info.flags & info.NEW_HEAD:
                            continue
                        success = False
                        print(info.summary)

                if success:
                    return
            except GitCommandError as err:
                print(err)

        if push_tags:
            print("Pushing all tags failed after %d attempts" % (self.PUSH_RETRY_LIMIT))
        elif ref is not None:
            print("Pushing %s failed after %d attempts" % (ref, self.PUSH_RETRY_LIMIT))
            print("For more information about the cause of this error, run the following command from the local repo: 'git push %s %s'" % (self.remote_name, ref))
        else:
            print("Pushing all branches failed after %d attempts" % (self.PUSH_RETRY_LIMIT))
        sys.exit(1)

    def get_remote_migration_tags(self):
        remote_tags_output = self.local_repo.git.ls_remote(self.remote_name, tags=True).split('\n')
        self.remote_migration_tags = dict((tag.split()[1].replace("refs/tags/",""), tag.split()[0]) for tag in remote_tags_output if self.MIGRATION_TAG_PREFIX in tag)

    def clean_up(self, clean_up_remote=False):
        tags = [tag for tag in self.local_repo.tags if tag.name.startswith(self.MIGRATION_TAG_PREFIX)]

        # delete the local tags
        TagReference.delete(self.local_repo, *tags)

        # delete the remote tags
        if clean_up_remote:
            tags_to_delete = [":" + tag_name for tag_name in self.remote_migration_tags]
            self.remote_repo.push(refspec=tags_to_delete)

parser = OptionParser()
parser.add_option("-l", "--local",
                  action="store", dest="localrepo", default=os.getcwd(),
                  help="The path to the local repo. If this option is not specified, the script will attempt to use current directory by default. If it is not a local git repo, the script will fail.")
parser.add_option("-r", "--remote",
                  action="store", dest="remoterepo", default="codecommit",
                  help="The name of the remote repository to be used as the push or migration destination. The remote must already be set in the local repo ('git remote add ...'). If this option is not specified, the script will use 'codecommit' by default.")
parser.add_option("-b", "--batch",
                  action="store", dest="batchsize", default="1000",
                  help="Specifies the commit batch size for pushes. If not explicitly set, the default is 1,000 commits.")
parser.add_option("-c", "--clean",
                  action="store_true", dest="clean", default=False,
                  help="Remove the temporary tags created by migration from both the local repo and the remote repository. This option will not do any migration work, just cleanup. Cleanup is done automatically at the end of a successful migration, but not after a failure so that when you re-run the script, the tags from the prior run can be used to identify commit batches that were not pushed successfully.")

(options, args) = parser.parse_args()

migration = RepositoryMigration()
migration.migrate_repository_in_parts(options.localrepo, options.remoterepo, int(options.batchsize), options.clean)
```