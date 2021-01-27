# gitcommit 数据安全性

[TOC]

## AWS [责任共担模式](http://aws.amazon.com/compliance/shared-responsibility-model/)[数据隐私常见问题](http://aws.amazon.com/compliance/data-privacy-faq) 有关欧洲数据保护的信息，请参阅 *AWS 安全性博客* 上的 [AWS 责任共担模式和 GDPR](http://aws.amazon.com/blogs/security/the-aws-shared-responsibility-model-and-gdpr/)

出于数据保护目的，我们建议您保护 AWS 账户凭证并使用 AWS Identity and Access Management (IAM) 设置单独的用户账户。这仅向每个用户授予履行其工作职责所需的权限。我们还建议您通过以下方式保护您的数据：

- 对每个账户使用 Multi-Factor Authentication (MFA)。
- 使用 SSL/TLS 与 AWS 资源进行通信。建议使用 TLS 1.2 或更高版本。
- 使用 AWS CloudTrail 设置 API 和用户活动日志记录。
- 使用 AWS 加密解决方案以及 AWS 服务中的所有默认安全控制。
- 使用高级托管安全服务（例如 Amazon Macie），它有助于发现和保护存储在 Amazon S3 中的个人数据。
- 如果在通过命令行界面或 API 访问 AWS 时需要经过 FIPS 140-2 验证的加密模块，请使用 FIPS 终端节点。有关可用的 FIPS 终端节点的更多信息，请参阅[美国联邦信息处理标准 (FIPS) 第 140-2 版](http://aws.amazon.com/compliance/fips/)                                      

我们强烈建议您切勿将敏感的可识别信息（例如您客户的账号）放入自由格式字段（例如 **Name (名称)** 字段）。这包括使用控制台、API、AWS CLI 或 AWS 开发工具包处理 CodeCommit 或其他 AWS 服务时。您输入到 CodeCommit 或其他服务中的任何数据都可能被选取以包含在诊断日志中。当您向外部服务器提供                                     URL 时，请勿在 URL 中包含凭证信息来验证您对该服务器的请求。                                  

CodeCommit 存储库会自动执行静态加密。无需客户操作。CodeCommit 也会对传输中的存储库数据进行加密。您可以将 HTTPS 协议和/或 SSH 协议与                                     CodeCommit 存储库结合使用。有关更多信息，请参阅[对 AWS CodeCommit 进行设置 ](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/setting-up.html)。您还可以配置对 [ 存储库的](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/cross-account.html)跨账户访问CodeCommit。                                  

**主题**

- [AWS Key Management Service 和 AWS CodeCommit 存储库加密](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/encryption.html)
- [正在连接到 AWS CodeCommit 具有旋转凭据的存储库](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/temporary-access.html)

## AWS Key Management Service 和 AWS CodeCommit 存储库加密
CodeCommit 存储库中的数据在传输和存储状态下是加密的。当将数据推送到 CodeCommit 存储库中（例如，通过调用 git push）时，CodeCommit 在将收到的数据存储到存储库中时对其进行加密。当从 CodeCommit 存储库拉取数据（例如，通过调用 git pull）时，CodeCommit 先解密数据，然后将其发送给调用方。上述过程假定与推送或拉取请求关联的 IAM 用户已经过 AWS 的身份验证。发送或接收的数据使用 HTTPS 或 SSH 加密网络协议进行传输。

首次在您的 CodeCommit 存储库 账户中的新 AWS 区域中创建 AWS 时，CodeCommit 会在 （） 中的同一 AWS 区域中创建 AWS托管CMK（aws/codecommit 密钥）。AWS Key Management ServiceAWS KMS此密钥仅由 CodeCommit 使用（aws/codecommit密钥）。它存储在您的 AWS 账户中。CodeCommit 使用您的 AWS 托管CMK 加密和解密该 账户中该区域内此存储库及所有其他 存储库中的数据。CodeCommitAWS
重要

CodeCommit 对默认 aws/codecommit 密钥执行以下 AWS KMS 操作。IAM 用户无需获得下述操作的显式权限，但也不得向该用户附加拒绝对 aws/codecommit 密钥执行这些操作的任何策略。具体来说，当您创建第一个存储库时，您的 IAM 用户不得将以下任何权限设置为 deny：

    "kms:Encrypt"
    "kms:Decrypt"
    "kms:ReEncrypt"
    "kms:GenerateDataKey"
    "kms:GenerateDataKeyWithoutPlaintext"
    "kms:DescribeKey"

要查看 CodeCommit 生成的 AWS 托管密钥的相关信息，请执行以下操作：

    登录 AWS 管理控制台并通过以下网址打开 AWS Key Management Service (AWS KMS) 控制台：https://console.aws.amazon.com/kms
    
    要更改 AWS 区域，请使用页面右上角的区域选择器。
    
    在服务导航窗格中，选择 AWS managed keys (AWS 托管密钥)。请确保您已登录到要在其中查看密钥的 AWS 区域。
    
    在加密密钥列表中，选择别名为 aws/codecommit 的 AWS 托管CMK。此时将显示有关 AWS 托管 CMK 的基本信息。

您无法更改或删除此 AWS 托管 CMK。您不能使用 中的托管CMK 来加密或解密 存储库中的数据。AWS KMSCodeCommit
如何使用加密算法加密存储库数据

CodeCommit 使用两种不同的方法来加密数据。6 MB 以下的单个 Git 对象使用 AES-GCM-256 进行加密，该方法提供数据完整性验证。单个 Blob 的介于 6 MB 和最大 2 GB 之间的对象使用 AES-CBC-256 进行加密。CodeCommit 始终验证加密上下文。
加密上下文。

与 AWS KMS 集成的每项服务都会为加密和解密操作指定加密上下文。加密上下文是 AWS KMS 检查数据完整性时使用的额外的身份验证信息。如果为加密操作指定了加密上下文，则也必须在解密操作中指定它。否则，解密将失败。CodeCommit 将 CodeCommit 存储库 ID 用于加密上下文。您可以使用 get-repository 命令或 CodeCommit 控制台来查找存储库 ID。在 AWS CloudTrail 日志中搜索 CodeCommit 存储库 ID，以了解对 AWS KMS 中的哪个密钥执行了加密操作来加密或解密 CodeCommit 存储库中的数据。

有关 AWS KMS 的更多信息，请参阅 AWS Key Management Service 开发人员指南。 

AWS Identity and Access Management (IAM) 是一项 AWS 服务，可帮助管理员安全地控制对 AWS 资源的访问。IAM 管理员控制谁可以通过身份验证 （登录）和授权 （具有权限）以使用 CodeCommit 资源。IAM 是一项无需额外费用即可使用的 AWS 服务。

主题

    Audience
    使用 身份进行身份验证
    使用策略管理访问权限
    AWS CodeCommit 的身份验证和访问控制
    AWS CodeCommit 如何与 IAM 协同工作
    CodeCommit 基于资源的策略
    基于 CodeCommit 标签的授权
    CodeCommit IAM 角色
    AWS CodeCommit 基于身份的策略示例
    排查 AWS CodeCommit 身份和权限的问题

Audience

如何使用 AWS Identity and Access Management (IAM) 因您可以在 CodeCommit 中执行的操作而异。

服务用户 – 如果您使用 CodeCommit 服务来完成工作，则您的管理员会为您提供所需的凭证和权限。当您使用更多 CodeCommit 功能来完成工作时，您可能需要额外权限。了解如何管理访问权限可帮助您向管理员请求适合的权限。如果您无法访问 CodeCommit 中的一项功能，请参阅排查 AWS CodeCommit 身份和权限的问题。

服务管理员 – 如果您在公司负责管理 CodeCommit 资源，则您可能具有 CodeCommit 的完全访问权限。您有责任确定您的员工应访问哪些 CodeCommit 功能和资源。然后，您必须向 IAM 管理员提交请求以更改您的服务用户的权限。检查此页上的信息，了解 IAM 的基本概念。要了解有关您的公司如何将 IAM 与 CodeCommit 搭配使用的更多信息，请参阅AWS CodeCommit 如何与 IAM 协同工作。

IAM 管理员 – 如果您是 IAM 管理员，您可能希望了解有关您可以如何编写策略以管理 CodeCommit 访问权限的详细信息。要查看您可在 IAM 中使用的基于身份的 CodeCommit 示例策略，请参阅AWS CodeCommit 基于身份的策略示例。
使用 身份进行身份验证

身份验证是您使用身份凭证登录 AWS 的方法。有关使用 AWS 管理控制台 登录的更多信息，请参阅 IAM 用户指南 中的 以 IAM 用户或 根用户身份登录 AWS 管理控制台。

您必须以 AWS 账户根用户、IAM 用户身份或通过代入 IAM 角色进行身份验证（登录到 AWS）。您还可以使用公司的单一登录身份验证方法，甚至使用 Google 或 Facebook 登录。在这些案例中，您的管理员以前使用 IAM 角色设置了联合身份验证。在您使用来自其他公司的凭证访问 AWS 时，您间接地代入了角色。

要直接登录到 AWS 管理控制台

，请使用您的密码和 根用户 电子邮件地址或 IAM 用户名。您可以使用 根用户 或 IAM 用户访问密钥以编程方式访问 AWS。AWS 提供了开发工具包和命令行工具，可使用您的凭证对您的请求进行加密签名。如果您不使用 AWS 工具，则必须自行对请求签名。使用签名版本 4（用于对入站 API 请求进行验证的协议）完成此操作。有关身份验证请求的更多信息，请参阅 AWS General Reference 中的签名版本 4 签名流程。

无论使用何种身份验证方法，您可能还需要提供其他安全信息。例如，AWS 建议您使用多重身份验证 (MFA) 来提高账户的安全性。要了解更多信息，请参阅 IAM 用户指南 中的在 AWS 中使用 Multi-Factor Authentication (MFA)。
AWS 账户根用户

当您首次创建 AWS 账户时，最初使用的是一个对账户中所有 AWS 服务和资源有完全访问权限的单点登录身份。此身份称为 AWS 账户 根用户，可使用您创建账户时所用的电子邮件地址和密码登录来获得此身份。强烈建议您不使用 根用户 执行日常任务，即使是管理任务。请遵守仅将 根用户 用于创建首个 IAM 用户的最佳实践。然后请妥善保存 根用户 凭证，仅用它们执行少数账户和服务管理任务。
IAM 用户和组

IAM 用户是 AWS 账户内对某个人员或应用程序具有特定权限的一个身份。IAM 用户可以拥有长期凭证，例如用户名和密码或一组访问密钥。要了解如何生成访问密钥，请参阅 IAM 用户指南 中的管理 IAM 用户的访问密钥。为 IAM 用户生成访问密钥时，请确保查看并安全保存密钥对。您以后无法找回秘密访问密钥，而是必须生成新的访问密钥对。

IAM 组 是指定一个 IAM 用户集合的身份。您不能使用组的身份登录。您可以使用组来一次性为多个用户指定权限。如果有大量用户，使用组可以更轻松地管理用户权限。例如，您有一个名为 IAMAdmins 的组并为该组授予管理 IAM 资源的权限。

用户与角色不同。用户唯一地与某个人员或应用程序关联，而角色旨在让需要它的任何人代入。用户具有永久的长期凭证，而角色提供临时凭证。要了解更多信息，请参阅 IAM 用户指南 中的何时创建 IAM 用户（而不是角色）。
IAM 角色

IAM 角色 是 AWS 账户中具有特定权限的实体。它类似于 IAM 用户，但未与特定人员关联。您可以通过切换角色，在 AWS 管理控制台中暂时代入 IAM 角色。您可以调用 AWS CLI 或 AWS API 操作或使用自定义 URL 以代入角色。有关使用角色方法的更多信息，请参阅 IAM 用户指南 中的使用 IAM 角色。

具有临时凭证的 IAM 角色在以下情况下很有用：

    临时 IAM 用户权限 – IAM 用户可代入 IAM 角色，暂时获得针对特定任务的不同权限。
    
    联合身份用户访问 – 您也可以不创建 IAM 用户，而是使用来自 AWS Directory Service、您的企业用户目录或 Web 身份提供商的现有身份。这些用户被称为联合身份用户。在通过身份提供商请求访问权限时，AWS 将为联合身份用户分配角色。有关联合身份用户的更多信息，请参阅 IAM 用户指南 中的联合身份用户和角色。
    
    跨账户访问 – 您可以使用 IAM 角色允许其他账户中的某个人（可信任委托人）访问您账户中的资源。角色是授予跨账户访问权限的主要方式。但是，对于某些 AWS 服务，您可以将策略直接附加到资源（而不是使用角色作为代理）。要了解用于跨账户访问的角色和基于资源的策略之间的差别，请参阅 IAM 用户指南 中的 IAM 角色与基于资源的策略有何不同。
    
    跨服务访问 – Some AWS services use features in other AWS services. For example, when you make a call in a service, it's common for that service to run applications in Amazon EC2 or store objects in Amazon S3. A service might do this using the calling principal's permissions, using a service role, or using a service-linked role.
    
        委托人权限 – When you use an IAM user or role to perform actions in AWS, you are considered a principal. Policies grant permissions to a principal. When you use some services, you might perform an action that then triggers another action in a different service. In this case, you must have permissions to perform both actions. To see whether an action requires additional dependent actions in a policy, see Actions, Resources, and Condition Keys for AWS CodeCommit in the Service Authorization Reference.
    
        服务角色 – 服务角色是服务代入以代表您执行操作的 IAM 角色。服务角色只在您的账户内提供访问权限，不能用于为访问其他账户中的服务授权。IAM 管理员可以在 IAM 中创建、修改和删除服务角色。有关更多信息，请参阅 IAM 用户指南 中的创建角色以向 AWS 服务委派权限。
    
        服务相关角色 – A service-linked role is a type of service role that is linked to an AWS service. The service can assume the role to perform an action on your behalf. Service-linked roles appear in your IAM account and are owned by the service. An IAM administrator can view, but not edit the permissions for service-linked roles.
    
    在 Amazon EC2 上运行的应用程序–对于在 EC2 实例上运行、并发出 AWS CLI 或 AWS API 请求的应用程序，您可以使用 IAM 角色管理它们的临时凭证。这优先于在 EC2 实例中存储访问密钥。要将 AWS 角色分配给 EC2 实例并使其对该实例的所有应用程序可用，您可以创建一个附加到实例的实例配置文件。实例配置文件包含角色，并使 EC2 实例上运行的程序能够获得临时凭证。有关更多信息，请参阅 IAM 用户指南 中的使用 IAM 角色向在 Amazon EC2 实例上运行的应用程序授予权限。

要了解是否使用 IAM 角色或 IAM 用户，请参阅 IAM 用户指南 中的何时创建 IAM 角色（而不是用户）。
使用策略管理访问权限

您将创建策略并将其附加到 IAM 身份或 AWS 资源，以便控制 AWS 中的访问。策略是 AWS中的对象；在与标识或资源相关联时，策略定义它们的权限。您可以通过 根用户 用户或 IAM 用户身份登录，也可以代入 IAM 角色。随后，当您提出请求时，AWS 会评估相关的基于身份或基于资源的策略。策略中的权限确定是允许还是拒绝请求。大多数策略在 AWS 中存储为 JSON 文档。有关 JSON 策略文档的结构和内容的更多信息，请参阅 IAM 用户指南 中的 JSON 策略概述。

Administrators can use AWS JSON policies to specify who has access to what. That is, which principal can perform actions on what resources, and under what conditions.

每个 IAM 实体（用户或角色）在一开始都没有权限。换言之，默认情况下，用户什么都不能做，甚至不能更改他们自己的密码。要为用户授予执行某些操作的权限，管理员必须将权限策略附加到用户。或者，管理员可以将用户添加到具有预期权限的组中。当管理员为某个组授予访问权限时，该组内的全部用户都会获得这些访问权限。

IAM 策略定义操作的权限，无论您使用哪种方法执行操作。例如，假设您有一个允许 iam:GetRole 操作的策略。具有该策略的用户可以从 AWS 管理控制台、AWS CLI 或 AWS API 获取角色信息。
基于身份的策略

Identity-based policies are JSON permissions policy documents that you can attach to an identity, such as an IAM user, group of users, or role. These policies control what actions users and roles can perform, on which resources, and under what conditions. To learn how to create an identity-based policy, see Creating IAM policies in the IAM 用户指南.

基于身份的策略可以进一步归类为内联策略或托管策略。内联策略直接嵌入单个用户、组或角色中。托管策略是可以附加到 AWS 账户中的多个用户、组和角色的独立策略。托管策略包括 AWS 托管策略和客户托管策略。要了解如何在托管策略或内联策略之间进行选择，请参阅 IAM 用户指南 中的在托管策略与内联策略之间进行选择 。
基于资源的策略

Resource-based policies are JSON policy documents that you attach to a resource. Examples of resource-based policies are IAM role trust policies and Amazon S3 bucket policies. In services that support resource-based policies, service administrators can use them to control access to a specific resource. For the resource where the policy is attached, the policy defines what actions a specified principal can perform on that resource and under what conditions. You must specify a principal in a resource-based policy. Principals can include accounts, users, roles, federated users, or AWS services.

基于资源的策略是位于该服务中的内联策略。您不能在基于资源的策略中使用来自 IAM 的 AWS 托管策略。
访问控制列表 (ACL)

Access control lists (ACLs) control which principals (account members, users, or roles) have permissions to access a resource. ACLs are similar to resource-based policies, although they do not use the JSON policy document format.

Amazon S3、AWS WAF 和 Amazon VPC 是支持 ACL 的服务示例。要了解有关 ACL 的更多信息，请参阅Amazon Simple Storage Service 开发人员指南 中的访问控制列表 (ACL) 概述。
其他策略类型

AWS 支持额外的、不太常用的策略类型。这些策略类型可以设置更常用的策略类型向您授予的最大权限。

    权限边界 – 权限边界是一项高级功能，借助该功能，您可以设置基于身份的策略可以授予 IAM 实体的最大权限（IAM 用户或角色）。您可为实体设置权限边界。这些结果权限是实体的基于身份的策略及其权限边界的交集。在 Principal 中指定用户或角色的基于资源的策略不受权限边界限制。任一项策略中的显式拒绝将覆盖允许。有关权限边界的更多信息，请参阅 IAM 用户指南 中的 IAM 实体的权限边界。
    
    服务控制策略 (SCP) – SCP 是 JSON 策略，指定了组织或组织单位 (OU) 在 AWS Organizations 中的最大权限。AWS Organizations 是一项服务，用于分组和集中管理您的企业拥有的多个 AWS 账户。如果在组织内启用了所有功能，则可对任意或全部账户应用服务控制策略 (SCP)。SCP 限制成员账户中实体（包括每个 AWS 账户根用户）的权限。有关 组织 和 SCP 的更多信息，请参阅 AWS Organizations 用户指南 中的 SCP 工作原理。
    
    会话策略 – 会话策略是当您以编程方式为角色或联合身份用户创建临时会话时作为参数传递的高级策略。结果会话的权限是用户或角色的基于身份的策略和会话策略的交集。权限也可以来自基于资源的策略。任一项策略中的显式拒绝将覆盖允许。有关更多信息，请参阅 IAM 用户指南 中的会话策略。

多个策略类型

当多个类型的策略应用于一个请求时，生成的权限更加复杂和难以理解。要了解 AWS 如何确定在涉及多个策略类型时是否允许请求，请参阅 IAM 用户指南 中的策略评估逻辑。
CodeCommit 基于资源的策略

CodeCommit 不支持基于资源的策略。
基于 CodeCommit 标签的授权

您可以将标签附加到 CodeCommit 资源或将请求中的标签传递到 CodeCommit。要基于标签控制访问，您需要使用 codecommit:ResourceTag/key-name、aws:RequestTag/key-name 或 aws:TagKeys 条件键在策略的条件元素中提供标签信息。有关标记 CodeCommit 资源的更多信息，请参阅示例 5：拒绝或允许对具有标签的存储库执行操作。有关标记策略的更多信息，请参阅标记 AWS 资源。

CodeCommit 还支持基于会话标签的策略。有关更多信息，请参阅会话标签。
使用标记在中提供身份信息 CodeCommit

CodeCommit 支持使用会话标签，这些标签是您在代入 IAM 角色、使用临时凭证或在 AWS Security Token Service (AWS STS) 中对用户进行联合身份验证时传递的键值对属性。您还可以将标签与 IAM 用户关联。您可以使用这些标签中提供的信息，以便更轻松地识别谁进行了更改或导致了事件。CodeCommit 在 CodeCommit 事件中包含具有以下键名称的标签的值：
键名称 	Value
displayName 	要显示并要与用户关联的便于阅读的名称（例如 Mary Major 或 Saanvi Sarkar）。
emailAddress 	您希望为用户显示并与用户关联的电子邮件地址（例如 mary_major@example.com 或 saanvi_sarkar@example.com）。

如果提供此信息，CodeCommit 将其包括在发送到 Amazon EventBridge 和 Amazon CloudWatch Events 的事件中。有关更多信息，请参阅监控 CodeCommit 事件 Amazon EventBridge 和 Amazon CloudWatch Events。

要使用会话标记,角色必须具有包含 sts:TagSession 权限设置为 Allow。如果您正在使用联合访问,则可以在设置中配置显示名称和电子邮件标签信息。例如，如果您使用的是 Azure Active Directory，则可能会提供以下声明信息：
声明名称 	Value
https://aws.amazon.com/SAML/Attributes/PrincipalTag:displayName 	user.displayname
https://aws.amazon.com/SAML/Attributes/PrincipalTag:emailAddress 	user.mail

您可以使用 AWS CLI 以传递会话标记 displayName 和 emailAddress 使用 AssumeRole。例如, 想要承担名为的角色的用户 Developer 想要将她的名字 Mary Major 可能会使用 assume-role 命令类似于以下内容:

aws sts assume-role \
--role-arn arn:aws:iam::123456789012:role/Developer \
--role-session-name Mary-Major \
–-tags Key=displayName,Value="Mary Major" Key=emailAddress,Value="mary_major@example.com" \
--external-id Example987

有关更多信息,请参阅 AssumeRole.

您可以使用 AssumeRoleWithSAML 操作返回包含 displayName 和 emailAddress 标签的一组临时凭证。您可以在访问 CodeCommit 存储库时使用这些标签。这要求您的公司或组织已将您的第三方 SAML 解决方案与 AWS 集成。如果是这样，则可以将 SAML 属性作为会话标签传递。例如,如果您想传递显示名称和电子邮件地址的名称为 Saanvi Sarkar 作为会话标记:

<Attribute Name="https://aws.amazon.com/SAML/Attributes/PrincipalTag:displayName">
  <AttributeValue>Saanvi Sarkar</AttributeValue>
</Attribute>
<Attribute Name="https://aws.amazon.com/SAML/Attributes/PrincipalTag:emailAddress">
  <AttributeValue>saanvi_sarkar@example.com</AttributeValue>
</Attribute>

有关更多信息,请参阅 使用传递会话标记 AssumeRoleWithSAML.

您可以使用 AssumeRoleWithIdentity 操作返回包含 displayName 和 emailAddress 标签的一组临时凭证。您可以在访问 CodeCommit 存储库时使用这些标签。要从传递会话标记 OpenID 连接(OIDC),您必须在JSONWeb令牌(JWT)中包含会话标记。例如,用于呼叫 AssumeRoleWithWebIdentity 包括 displayName 和 emailAddress 会话标记 Li Juan:

{
    "sub": "lijuan",
    "aud": "ac_oic_client",
    "jti": "ZYUCeREXAMPLE",
    "iss": "https://xyz.com",
    "iat": 1566583294,
    "exp": 1566583354,
    "auth_time": 1566583292,
    "https://aws.amazon.com/tags": {
        "principal_tags": {
            "displayName": ["Li Juan"],
            "emailAddress": ["li_juan@example.com"],
        },
        "transitive_tag_keys": [
            "displayName",
            "emailAddress"
        ]
    }
}

有关更多信息,请参阅 使用传递会话标记 AssumeRoleWithWebIdentity.

您可以使用 GetFederationToken 操作返回包含 displayName 和 emailAddress 标签的一组临时凭证。您可以在访问 CodeCommit 存储库时使用这些标签。例如，要使用 AWS CLI 获取包含 displayName 和 emailAddress 标签的联合令牌：

aws sts get-federation-token \
--name my-federated-user \
–-tags key=displayName,value="Nikhil Jayashankar" key=emailAddress,value=nikhil_jayashankar@example.com

有关更多信息,请参阅 使用传递会话标记 GetFederationToken.
CodeCommit IAM 角色

IAM 角色是 AWS 账户中具有特定权限的实体。
将临时凭证用于 CodeCommit

您可以使用临时凭证进行联合身份登录，担任 IAM 角色或担任跨账户角色。您可通过拨打以下电话获取临时安全凭证: AWS STS API操作,如 AssumeRole 或 GetFederationToken.

CodeCommit 支持使用临时凭证。有关更多信息，请参阅正在连接到 AWS CodeCommit 具有旋转凭据的存储库。
服务相关角色

服务相关角色允许 AWS 服务访问其他服务中的资源以代表您完成操作。服务相关角色显示在您的 IAM 账户中，并由该服务拥有。IAM 管理员可以查看但不能编辑服务相关角色的权限。

CodeCommit 不使用服务相关角色。
服务角色

此功能允许服务代表您担任服务角色。此角色允许服务访问其他服务中的资源以代表您完成操作。服务角色显示在您的 IAM 账户中，并由该账户拥有。这意味着 IAM 管理员可以更改此角色的权限。但是，这样操作可能会中断服务的功能。

CodeCommit 不使用服务角色。
AWS CodeCommit 基于身份的策略示例

默认情况下，IAM 用户和角色没有创建或修改 CodeCommit 资源的权限。它们还无法使用 AWS 管理控制台、AWS CLI 或 AWS API 执行任务。IAM 管理员必须创建 IAM 策略，为用户和角色授予权限，以便对他们所需的指定资源执行特定的 API 操作。然后，管理员必须将这些策略附加到需要这些权限的 IAM 用户或组。

有关策略示例，请参阅以下主题：

    示例 1：允许用户在单个 AWS 区域中执行 CodeCommit 操作
    
    示例 2：允许用户对单个存储库使用 Git
    
    示例 3：允许从指定 IP 地址范围连接的用户访问存储库
    
    示例 4：拒绝或允许对分支执行操作
    
    示例 5：拒绝或允许对具有标签的存储库执行操作
    
    配置跨帐户访问 AWS CodeCommit 使用角色的存储库

了解如何创建 IAM 使用这些示例JSON策略文档的基于身份的策略,请参阅 在JSON选项卡上创建策略 在 IAM 用户指南.

主题

    策略最佳实践
    使用 CodeCommit 控制台
    允许用户查看他们自己的权限
    查看CodeCommit repositories 基于标签

策略最佳实践

基于身份的策略非常强大。它们确定某个人是否可以创建、访问或删除您账户中的 CodeCommit 资源。这些操作可能会使 AWS 账户产生成本。创建或编辑基于身份的策略时，请遵循以下准则和建议：

    开始使用 AWS 托管策略 – 要快速开始使用 CodeCommit，请使用 AWS 托管策略，为您的员工授予他们所需的权限。这些策略已在您的账户中提供，并由 AWS 维护和更新。有关更多信息，请参阅 IAM 用户指南 中的利用 AWS 托管策略开始使用权限。
    
    授予最低权限 – 创建自定义策略时，仅授予执行任务所需的许可。最开始只授予最低权限，然后根据需要授予其他权限。这样做比起一开始就授予过于宽松的权限而后再尝试收紧权限来说更为安全。有关更多信息，请参阅 IAM 用户指南 中的授予最小权限。
    
    为敏感操作启用 MFA – 为增强安全性，要求 IAM 用户使用多重身份验证 (MFA) 来访问敏感资源或 API 操作。有关更多信息，请参阅 IAM 用户指南 中的在 AWS 中使用多重身份验证 (MFA)。
    
    使用策略条件来增强安全性 – 在切实可行的范围内，定义基于身份的策略在哪些情况下允许访问资源。例如，您可编写条件来指定请求必须来自允许的 IP 地址范围。您也可以编写条件，以便仅允许指定日期或时间范围内的请求，或者要求使用 SSL 或 MFA。有关更多信息，请参阅 IAM 用户指南 中的 IAM JSON 策略元素：Condition。

使用 CodeCommit 控制台

要访问 AWS CodeCommit 控制台，您必须拥有一组最低的权限。这些权限必须允许您列出和查看有关您的 AWS 账户中的 CodeCommit 资源的详细信息。如果创建比必需的最低权限更为严格的基于身份的策略，对于附加了该策略的实体（IAM 用户或角色），控制台将无法按预期正常运行。

要确保这些实体仍可使用 CodeCommit 控制台，也可向实体附加以下 AWS 托管策略。有关更多信息,请参阅 为用户添加权限 在 IAM 用户指南:

有关更多信息，请参阅对 IAM 使用基于身份的策略（CodeCommit 策略）。

对于仅调用 AWS CLI 或 AWS API 的用户，您不需要允许最低控制台权限。相反，只允许访问与您尝试执行的 API 操作相匹配的操作。
允许用户查看他们自己的权限

此示例显示您可以如何创建策略，以便允许 IAM 用户查看附加到其用户身份的内联和托管策略。此策略包括在控制台上完成此操作或者以编程方式使用 AWS CLI 或 AWS API 所需的权限。

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ViewOwnUserInfo",
            "Effect": "Allow",
            "Action": [
                "iam:GetUserPolicy",
                "iam:ListGroupsForUser",
                "iam:ListAttachedUserPolicies",
                "iam:ListUserPolicies",
                "iam:GetUser"
            ],
            "Resource": ["arn:aws:iam::*:user/${aws:username}"]
        },
        {
            "Sid": "NavigateInConsole",
            "Effect": "Allow",
            "Action": [
                "iam:GetGroupPolicy",
                "iam:GetPolicyVersion",
                "iam:GetPolicy",
                "iam:ListAttachedGroupPolicies",
                "iam:ListGroupPolicies",
                "iam:ListPolicyVersions",
                "iam:ListPolicies",
                "iam:ListUsers"
            ],
            "Resource": "*"
        }
    ]
}

查看CodeCommit repositories 基于标签

您可以在基于身份的策略中使用条件，以便基于标签控制对 CodeCommit 资源的访问。有关演示如何执行此操作的示例策略，请参阅示例 5：拒绝或允许对具有标签的存储库执行操作。

有关更多信息,请参阅 IAM JSON策略元素: 条件 在 IAM 用户指南.
排查 AWS CodeCommit 身份和权限的问题

使用以下信息可帮助您诊断和修复在使用 CodeCommit 和 IAM 时可能遇到的常见问题。

- 主题

    我无权在 CodeCommit 中执行操作
    我无权执行 iam:PassRole
    我想要查看我的访问密钥
    我是管理员，并且想要允许其他人访问 CodeCommit
    我希望允许我的 AWS 账户之外的人员访问我的 CodeCommit 资源

- 我无权在 CodeCommit 中执行操作

如果 AWS 管理控制台 告诉您，您无权执行某个操作，则必须联系您的管理员寻求帮助。您的管理员是指为您提供用户名和密码的那个人。

有关更多信息，请参阅使用 CodeCommit 控制台所需的权限
我无权执行 iam:PassRole

如果您收到错误消息，提示您无权执行 iam:PassRole 操作，则必须联系您的管理员寻求帮助。您的管理员是指为您提供用户名和密码的那个人。请求那个人更新您的策略，以便允许您将角色传递给 CodeCommit。

有些 AWS 服务允许您将现有角色传递到该服务，而不是创建新服务角色或服务相关角色。为此，您必须具有将角色传递到服务的权限。

当名为 marymajor 的 IAM 用户尝试使用控制台在 CodeCommit 中执行操作时，会发生以下示例错误。但是，服务必须具有服务角色所授予的权限才可执行操作。Mary 不具有将角色传递到服务的权限。

User: arn:aws:iam::123456789012:user/marymajor is not authorized to perform: iam:PassRole

在这种情况下，Mary 请求她的管理员来更新其策略，以允许她执行 iam:PassRole 操作。
我想要查看我的访问密钥

创建 IAM 用户访问密钥之后，您可以随时查看您的访问密钥 ID。但是，您无法再查看您的秘密访问密钥。如果您丢失了私有密钥，则必须创建一个新的访问密钥对。

访问密钥包含两部分：访问密钥 ID（例如 AKIAIOSFODNN7EXAMPLE）和秘密访问密钥（例如 wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY）。与用户名和密码一样，您必须同时使用访问密钥 ID 和秘密访问密钥对请求执行身份验证。像对用户名和密码一样，安全地管理访问密钥。
重要

请不要向第三方提供访问密钥，甚至为了帮助找到您的规范用户 ID 也不能提供。如果您这样做，可能会向某人提供对您的账户的永久访问权限。

当您创建访问密钥对时，系统会提示您将访问密钥 ID 和秘密访问密钥保存在一个安全位置。秘密访问密钥仅在您创建它时可用。如果您丢失了秘密访问密钥，则必须向您的 IAM 用户添加新的访问密钥。您最多可拥有两个访问密钥。如果您已有两个密钥，则必须删除一个密钥对，然后再创建新的密钥。要查看说明，请参阅 IAM 用户指南 中的管理访问密钥。
我是管理员，并且想要允许其他人访问 CodeCommit

要允许其他人访问 CodeCommit，您必须为需要访问权限的人员或应用程序创建 IAM 实体（用户或角色）。他们（它们）将使用该实体的凭证访问 AWS。然后，您必须将策略附加到实体，以便在 CodeCommit 中为他们（它们）授予正确的权限。

要立即开始使用，请参阅 IAM 用户指南 中的创建您的第一个 IAM 委托用户和组。
我希望允许我的 AWS 账户之外的人员访问我的 CodeCommit 资源

有关更多信息，请参阅配置跨帐户访问 AWS CodeCommit 使用角色的存储库



- 以下基于身份的策略示例说明账户管理员如何将权限策略附加到 IAM 身份（即用户、组和角色），从而授予对 CodeCommit 资源执行操作的权限。

  ​                                        

  重要

  我们建议您首先阅读以下介绍性主题，这些主题说明了可用于管理 CodeCommit 资源访问的基本概念和选项。有关更多信息，请参阅[管理 CodeCommit 资源的访问权限的概述](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control.html#auth-and-access-control-iam-access-control-identity-based)。                                        

  **主题**

  - [使用 CodeCommit 控制台所需的权限](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#console-permissions)
  - [在控制台中查看资源](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#console-resources)
  - [适用于 CodeCommit 的 AWS 托管（预定义）策略](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#managed-policies)
  - [客户管理的策略示例](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#customer-managed-policies)

  下面是基于身份的权限策略的示例：

  ```
  
  {
    "Version": "2012-10-17",
    "Statement" : [
      {
        "Effect" : "Allow",
        "Action" : [
          "codecommit:BatchGetRepositories"
        ],
        "Resource" : [
          "arn:aws:codecommit:us-east-2:111111111111:MyDestinationRepo",
          "arn:aws:codecommit:us-east-2:111111111111:MyDemo*"
        ]
      }
    ]
  }
  ```

  此策略有一条语句允许用户获取有关 `us-east-2` 区域中名为 `MyDestinationRepo` 的 CodeCommit 存储库以及名称以 `MyDemo` 开头的所有 CodeCommit 存储库的信息。                                  

  ## 使用 CodeCommit 控制台所需的权限

  要查看每个 CodeCommit API 操作所需的权限以及有关 CodeCommit 操作的更多信息，请参阅[CodeCommit 权限参考](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-permissions-reference.html)。                                  

  要允许用户使用 CodeCommit 控制台，管理员必须授予他们执行 CodeCommit 操作的权限。例如，您可以将 [AWSCodeCommitPowerUser](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#managed-policies-poweruser) 托管策略或其等效策略附加到用户或组。                                  

  除了通过基于身份的策略授予用户的权限外，CodeCommit 还需要执行 AWS Key Management Service (AWS KMS) 操作的权限。IAM                                     用户不需要这些操作的显式 `Allow` 权限，但用户不能附加任何将以下权限设为 `Deny` 的策略：                                  

  ```
  
          "kms:Encrypt",
          "kms:Decrypt",
          "kms:ReEncrypt",
          "kms:GenerateDataKey",
          "kms:GenerateDataKeyWithoutPlaintext",
          "kms:DescribeKey"
  ```

  有关加密和 CodeCommit 的更多信息，请参阅[AWS KMS 和加密](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/encryption.html)。                                  

  ## 在控制台中查看资源

  CodeCommit 控制台需要 `ListRepositories` 权限，以显示您的 AWS 账户在所登录的 AWS 区域中的存储库列表。该控制台还包括一个 **Go to resource (转到资源)** 功能，可对资源快速执行不区分大小写的搜索。此搜索通过您的 AWS 账户，在您登录的 AWS 区域中执行。将显示以下服务中的以下资源：                                  

  - AWS CodeBuild：构建项目
  - AWS CodeCommit：存储库
  - AWS CodeDeploy：应用程序
  - AWS CodePipeline：管道

  要在所有服务中跨资源执行此搜索，您必须具有如下权限：

  - CodeBuild: `ListProjects`
  - CodeCommit: `ListRepositories`
  - CodeDeploy: `ListApplications`
  - CodePipeline: `ListPipelines`

  如果您没有针对某个服务的权限，搜索将不会针对该服务的资源返回结果。即使您有权限查看资源，但如果特定资源明确 `Deny` 查看，搜索也不会返回这些资源。                                  

  ## 适用于 CodeCommit 的 AWS 托管（预定义）策略

  AWS 通过提供由 AWS 创建和管理的独立 IAM  策略来解决许多常用案例。这些 AWS 托管策略将授予针对常用案例的必要权限。CodeCommit 的托管策略还提供在其他服务（如                                     IAM、Amazon SNS 和 Amazon CloudWatch  Events）中执行操作的权限，这是授予了相关策略的用户职责所必需的。例如，AWSCodeCommitFullAccess                                     策略是管理级用户策略，允许具有此策略的用户为存储库创建和管理  CloudWatch Events 规则（名称前缀为 `codecommit` 的规则），并为存储库相关事件通知创建和管理 Amazon SNS 主题（名称前缀为 `codecommit` 的主题），以及在 CodeCommit 中管理存储库。                                  

  以下 AWS 托管策略（可附加到账户中的用户）特定于 CodeCommit：

  **主题**

  - [AWSCodeCommitFullAccess](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#managed-policies-full)
  - [AWSCodeCommitPowerUser](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#managed-policies-poweruser)
  - [AWSCodeCommitReadOnly](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#managed-policies-read)
  - [CodeCommit managed policies and                                              notifications](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#notifications-permissions)
  - [AWS CodeCommit 托管策略和 Amazon CodeGuru Reviewer](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#codeguru-permissions)

  ### AWSCodeCommitFullAccess

  **AWSCodeCommitFullAccess** – 授予 CodeCommit 的完全访问权限。仅将此策略应用于您希望向其授予对 CodeCommit 存储库及您的 AWS 账户中的相关资源的完全控制权限（包括删除存储库的能力）的管理级用户。                                  

  AWSCodeCommitFullAccess 策略包含以下策略语句：

  ```
  
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": [
              "codecommit:*"
            ],
            "Resource": "*"
          },
          {
            "Sid": "CloudWatchEventsCodeCommitRulesAccess",
            "Effect": "Allow",
            "Action": [
              "events:DeleteRule",
              "events:DescribeRule",
              "events:DisableRule",
              "events:EnableRule",
              "events:PutRule",
              "events:PutTargets",
              "events:RemoveTargets",
              "events:ListTargetsByRule"
            ],
            "Resource": "arn:aws:events:*:*:rule/codecommit*"
          },
          {
            "Sid": "SNSTopicAndSubscriptionAccess",
            "Effect": "Allow",
            "Action": [
              "sns:CreateTopic",
              "sns:DeleteTopic",
              "sns:Subscribe",
              "sns:Unsubscribe",
              "sns:SetTopicAttributes"
            ],
            "Resource": "arn:aws:sns:*:*:codecommit*"
          },
          {
            "Sid": "SNSTopicAndSubscriptionReadAccess",
            "Effect": "Allow",
            "Action": [
              "sns:ListTopics",
              "sns:ListSubscriptionsByTopic",
              "sns:GetTopicAttributes"
            ],
            "Resource": "*"
          },
          {
            "Sid": "LambdaReadOnlyListAccess",
            "Effect": "Allow",
            "Action": [
              "lambda:ListFunctions"
            ],
            "Resource": "*"
          },
          {
            "Sid": "IAMReadOnlyListAccess",
            "Effect": "Allow",
            "Action": [
              "iam:ListUsers"
            ],
            "Resource": "*"
          },
          {
            "Sid": "IAMReadOnlyConsoleAccess",
            "Effect": "Allow",
            "Action": [
              "iam:ListAccessKeys",
              "iam:ListSSHPublicKeys",
              "iam:ListServiceSpecificCredentials"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
          },
          {
            "Sid": "IAMUserSSHKeys",
            "Effect": "Allow",
            "Action": [
              "iam:DeleteSSHPublicKey",
              "iam:GetSSHPublicKey",
              "iam:ListSSHPublicKeys",
              "iam:UpdateSSHPublicKey",
              "iam:UploadSSHPublicKey"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
          },
          {
            "Sid": "IAMSelfManageServiceSpecificCredentials",
            "Effect": "Allow",
            "Action": [
              "iam:CreateServiceSpecificCredential",
              "iam:UpdateServiceSpecificCredential",
              "iam:DeleteServiceSpecificCredential",
              "iam:ResetServiceSpecificCredential"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
          },
          {
            "Sid": "CodeStarNotificationsReadWriteAccess",
            "Effect": "Allow",
            "Action": [
              "codestar-notifications:CreateNotificationRule",
              "codestar-notifications:DescribeNotificationRule",
              "codestar-notifications:UpdateNotificationRule",
              "codestar-notifications:DeleteNotificationRule",
              "codestar-notifications:Subscribe",
              "codestar-notifications:Unsubscribe"
            ],
            "Resource": "*",
            "Condition": {
              "StringLike": {
                "codestar-notifications:NotificationsForResource": "arn:aws:codecommit:*"
              }
            }
          },
          {
            "Sid": "CodeStarNotificationsListAccess",
            "Effect": "Allow",
            "Action": [
              "codestar-notifications:ListNotificationRules",
              "codestar-notifications:ListTargets",
              "codestar-notifications:ListTagsforResource",
              "codestar-notifications:ListEventTypes"
            ],
            "Resource": "*"
          },
          {
            "Sid": "CodeStarNotificationsSNSTopicCreateAccess",
            "Effect": "Allow",
            "Action": [
              "sns:CreateTopic",
              "sns:SetTopicAttributes"
            ],
            "Resource": "arn:aws:sns:*:*:codestar-notifications*"
          },
          {
            "Sid": "AmazonCodeGuruReviewerFullAccess",
            "Effect": "Allow",
            "Action": [
              "codeguru-reviewer:AssociateRepository",
              "codeguru-reviewer:DescribeRepositoryAssociation",
              "codeguru-reviewer:ListRepositoryAssociations",
              "codeguru-reviewer:DisassociateRepository",
              "codeguru-reviewer:DescribeCodeReview",
              "codeguru-reviewer:ListCodeReviews"
            ],
            "Resource": "*"
          },
          {
            "Sid": "AmazonCodeGuruReviewerSLRCreation",
            "Action": "iam:CreateServiceLinkedRole",
            "Effect": "Allow",
            "Resource": "arn:aws:iam::*:role/aws-service-role/codeguru-reviewer.amazonaws.com/AWSServiceRoleForAmazonCodeGuruReviewer",
            "Condition": {
              "StringLike": {
                "iam:AWSServiceName": "codeguru-reviewer.amazonaws.com"
              }
            }
          },
          {
            "Sid": "CloudWatchEventsManagedRules",
            "Effect": "Allow",
            "Action": [
              "events:PutRule",
              "events:PutTargets",
              "events:DeleteRule",
              "events:RemoveTargets"
            ],
            "Resource": "*",
            "Condition": {
              "StringEquals": {
                "events:ManagedBy": "codeguru-reviewer.amazonaws.com"
              }
            }
          },
          {
            "Sid": "CodeStarNotificationsChatbotAccess",
            "Effect": "Allow",
            "Action": [
              "chatbot:DescribeSlackChannelConfigurations"
            ],
            "Resource": "*"
          }
        ]
      }
  ```

  ### AWSCodeCommitPowerUser

  **AWSCodeCommitPowerUser** – 允许用户访问 CodeCommit 和存储库相关资源的所有功能，但不允许删除 CodeCommit 存储库或在其他 AWS 服务（如 Amazon CloudWatch                                     Events）中创建或删除存储库相关资源。建议对大多数用户应用此策略。                                  

  AWSCodeCommitPowerUser 策略包含以下策略语句：

  ```
  
      {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Action": [
              "codecommit:AssociateApprovalRuleTemplateWithRepository",
              "codecommit:BatchAssociateApprovalRuleTemplateWithRepositories",
              "codecommit:BatchDisassociateApprovalRuleTemplateFromRepositories",
              "codecommit:BatchGet*",
              "codecommit:BatchDescribe*",
              "codecommit:Create*",
              "codecommit:DeleteBranch",
              "codecommit:DeleteFile",
              "codecommit:Describe*",
              "codecommit:DisassociateApprovalRuleTemplateFromRepository",
              "codecommit:EvaluatePullRequestApprovalRules",
              "codecommit:Get*",
              "codecommit:List*",
              "codecommit:Merge*",
              "codecommit:OverridePullRequestApprovalRules",
              "codecommit:Put*",
              "codecommit:Post*",
              "codecommit:TagResource",
              "codecommit:Test*",
              "codecommit:UntagResource",
              "codecommit:Update*",
              "codecommit:GitPull",
              "codecommit:GitPush"
            ],
            "Resource": "*"
          },
          {
            "Sid": "CloudWatchEventsCodeCommitRulesAccess",
            "Effect": "Allow",
            "Action": [
              "events:DeleteRule",
              "events:DescribeRule",
              "events:DisableRule",
              "events:EnableRule",
              "events:PutRule",
              "events:PutTargets",
              "events:RemoveTargets",
              "events:ListTargetsByRule"
            ],
            "Resource": "arn:aws:events:*:*:rule/codecommit*"
          },
          {
            "Sid": "SNSTopicAndSubscriptionAccess",
            "Effect": "Allow",
            "Action": [
              "sns:Subscribe",
              "sns:Unsubscribe"
            ],
            "Resource": "arn:aws:sns:*:*:codecommit*"
          },
          {
            "Sid": "SNSTopicAndSubscriptionReadAccess",
            "Effect": "Allow",
            "Action": [
              "sns:ListTopics",
              "sns:ListSubscriptionsByTopic",
              "sns:GetTopicAttributes"
            ],
            "Resource": "*"
          },
          {
            "Sid": "LambdaReadOnlyListAccess",
            "Effect": "Allow",
            "Action": [
              "lambda:ListFunctions"
            ],
            "Resource": "*"
          },
          {
            "Sid": "IAMReadOnlyListAccess",
            "Effect": "Allow",
            "Action": [
              "iam:ListUsers"
            ],
            "Resource": "*"
          },
          {
            "Sid": "IAMReadOnlyConsoleAccess",
            "Effect": "Allow",
            "Action": [
              "iam:ListAccessKeys",
              "iam:ListSSHPublicKeys",
              "iam:ListServiceSpecificCredentials"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
          },
          {
            "Sid": "IAMUserSSHKeys",
            "Effect": "Allow",
            "Action": [
              "iam:DeleteSSHPublicKey",
              "iam:GetSSHPublicKey",
              "iam:ListSSHPublicKeys",
              "iam:UpdateSSHPublicKey",
              "iam:UploadSSHPublicKey"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
          },
          {
            "Sid": "IAMSelfManageServiceSpecificCredentials",
            "Effect": "Allow",
            "Action": [
              "iam:CreateServiceSpecificCredential",
              "iam:UpdateServiceSpecificCredential",
              "iam:DeleteServiceSpecificCredential",
              "iam:ResetServiceSpecificCredential"
            ],
            "Resource": "arn:aws:iam::*:user/${aws:username}"
          },
          {
            "Sid": "CodeStarNotificationsReadWriteAccess",
            "Effect": "Allow",
            "Action": [
              "codestar-notifications:CreateNotificationRule",
              "codestar-notifications:DescribeNotificationRule",
              "codestar-notifications:UpdateNotificationRule",
              "codestar-notifications:Subscribe",
              "codestar-notifications:Unsubscribe"
            ],
            "Resource": "*",
            "Condition": {
              "StringLike": {
                "codestar-notifications:NotificationsForResource": "arn:aws:codecommit:*"
              }
            }
          },
          {
            "Sid": "CodeStarNotificationsListAccess",
            "Effect": "Allow",
            "Action": [
              "codestar-notifications:ListNotificationRules",
              "codestar-notifications:ListTargets",
              "codestar-notifications:ListTagsforResource",
              "codestar-notifications:ListEventTypes"
            ],
            "Resource": "*"
          },
          {
            "Sid": "AmazonCodeGuruReviewerFullAccess",
            "Effect": "Allow",
            "Action": [
              "codeguru-reviewer:AssociateRepository",
              "codeguru-reviewer:DescribeRepositoryAssociation",
              "codeguru-reviewer:ListRepositoryAssociations",
              "codeguru-reviewer:DisassociateRepository",
              "codeguru-reviewer:DescribeCodeReview",
              "codeguru-reviewer:ListCodeReviews"
            ],
            "Resource": "*"
          },
          {
            "Sid": "AmazonCodeGuruReviewerSLRCreation",
            "Action": "iam:CreateServiceLinkedRole",
            "Effect": "Allow",
            "Resource": "arn:aws:iam::*:role/aws-service-role/codeguru-reviewer.amazonaws.com/AWSServiceRoleForAmazonCodeGuruReviewer",
            "Condition": {
              "StringLike": {
                "iam:AWSServiceName": "codeguru-reviewer.amazonaws.com"
              }
            }
          },
          {
            "Sid": "CloudWatchEventsManagedRules",
            "Effect": "Allow",
            "Action": [
              "events:PutRule",
              "events:PutTargets",
              "events:DeleteRule",
              "events:RemoveTargets"
            ],
            "Resource": "*",
            "Condition": {
              "StringEquals": {
                "events:ManagedBy": "codeguru-reviewer.amazonaws.com"
              }
            }
          },
          {
            "Sid": "CodeStarNotificationsChatbotAccess",
            "Effect": "Allow",
            "Action": [
              "chatbot:DescribeSlackChannelConfigurations"
            ],
            "Resource": "*"
          }
        ]
      }
  ```

  ### AWSCodeCommitReadOnly

  **AWSCodeCommitReadOnly** – 授予对 CodeCommit 和其他 AWS 服务中的存储库相关资源的只读访问权限以及创建和管理自己的 CodeCommit 相关资源（如供其 IAM 用户在访问存储库时使用的                                     Git 凭证和 SSH 密钥）的能力。针对希望向其授予读取存储库内容的能力但不能对内容进行任何更改的用户，应用此策略。                                  

  AWSCodeCommitReadOnly 策略包含以下策略语句：

  ```
  
      { 
         "Version":"2012-10-17",
         "Statement":[ 
            { 
               "Effect":"Allow",
               "Action":[ 
                  "codecommit:BatchGet*",
                  "codecommit:BatchDescribe*",
                  "codecommit:Describe*",
                  "codecommit:EvaluatePullRequestApprovalRules",
                  "codecommit:Get*",
                  "codecommit:List*",
                  "codecommit:GitPull"
               ],
               "Resource":"*"
            },
            { 
               "Sid":"CloudWatchEventsCodeCommitRulesReadOnlyAccess",
               "Effect":"Allow",
               "Action":[ 
                  "events:DescribeRule",
                  "events:ListTargetsByRule"
               ],
               "Resource":"arn:aws:events:*:*:rule/codecommit*"
            },
            { 
               "Sid":"SNSSubscriptionAccess",
               "Effect":"Allow",
               "Action":[ 
                  "sns:ListTopics",
                  "sns:ListSubscriptionsByTopic",
                  "sns:GetTopicAttributes"
               ],
               "Resource":"*"
            },
            { 
               "Sid":"LambdaReadOnlyListAccess",
               "Effect":"Allow",
               "Action":[ 
                  "lambda:ListFunctions"
               ],
               "Resource":"*"
            },
            { 
               "Sid":"IAMReadOnlyListAccess",
               "Effect":"Allow",
               "Action":[ 
                  "iam:ListUsers"
               ],
               "Resource":"*"
            },
            { 
               "Sid":"IAMReadOnlyConsoleAccess",
               "Effect":"Allow",
               "Action":[ 
                  "iam:ListAccessKeys",
                  "iam:ListSSHPublicKeys",
                  "iam:ListServiceSpecificCredentials",
                  "iam:ListAccessKeys",
                  "iam:GetSSHPublicKey"
               ],
               "Resource":"arn:aws:iam::*:user/${aws:username}"
            },
            { 
               "Sid":"CodeStarNotificationsReadOnlyAccess",
               "Effect":"Allow",
               "Action":[ 
                  "codestar-notifications:DescribeNotificationRule"
               ],
               "Resource":"*",
               "Condition":{ 
                  "StringLike":{ 
                     "codestar-notifications:NotificationsForResource":"arn:aws:codecommit:*"
                  }
               }
            },
            { 
               "Sid":"CodeStarNotificationsListAccess",
               "Effect":"Allow",
               "Action":[ 
                  "codestar-notifications:ListNotificationRules",
                  "codestar-notifications:ListEventTypes",
                  "codestar-notifications:ListTargets"
               ],
               "Resource":"*"
            },
              {
            "Sid": "AmazonCodeGuruReviewerReadOnlyAccess",
            "Effect": "Allow",
            "Action": [
                  "codeguru-reviewer:DescribeRepositoryAssociation",
                  "codeguru-reviewer:ListRepositoryAssociations",
                  "codeguru-reviewer:DescribeCodeReview",
                  "codeguru-reviewer:ListCodeReviews"
            ],
            "Resource": "*"
          }
         ]
      }
  ```

  ### CodeCommit managed policies and                                     notifications                                  

  AWS CodeCommit supports notifications, which can notify users of important changes                                     to                                     repositories.                                      Managed policies for CodeCommit include policy statements for notification                                     functionality. For more information, see [What are                                        notifications?](https://docs.aws.amazon.com/codestar-notifications/latest/userguide/welcome.html).                                  

  #### Permissions related to notifications in full access                                     managed policies                                  

  The `AWSCodeCommitFullAccess` managed policy includes the following statements                                     to allow full access to notifications. Users with this managed policy applied can                                     also                                     create and manage Amazon SNS topics for notifications, subscribe and unsubscribe users                                     to                                     topics, list topics to choose as targets for notification rules, and list AWS Chatbot                                     clients configured for Slack.                                  

  ```
  
      {
          "Sid": "CodeStarNotificationsReadWriteAccess",
          "Effect": "Allow",
          "Action": [
              "codestar-notifications:CreateNotificationRule",
              "codestar-notifications:DescribeNotificationRule",
              "codestar-notifications:UpdateNotificationRule",
              "codestar-notifications:DeleteNotificationRule",
              "codestar-notifications:Subscribe",
              "codestar-notifications:Unsubscribe"
          ],
          "Resource": "*",
          "Condition" : {
              "StringLike" : {"codestar-notifications:NotificationsForResource" : "arn:aws:codecommit:*"} 
          }
      },    
      {
          "Sid": "CodeStarNotificationsListAccess",
          "Effect": "Allow",
          "Action": [
              "codestar-notifications:ListNotificationRules",
              "codestar-notifications:ListTargets",
              "codestar-notifications:ListTagsforResource,"
              "codestar-notifications:ListEventTypes"
          ],
          "Resource": "*"
      },
      {
          "Sid": "CodeStarNotificationsSNSTopicCreateAccess",
          "Effect": "Allow",
          "Action": [
              "sns:CreateTopic",
              "sns:SetTopicAttributes"
          ],
          "Resource": "arn:aws:sns:*:*:codestar-notifications*"
      },
      {
          "Sid": "CodeStarNotificationsChatbotAccess",
          "Effect": "Allow",
          "Action": [
              "chatbot:DescribeSlackChannelConfigurations"
            ],
         "Resource": "*"
      }
  ```

  #### Permissions related to notifications in read-only managed                                     policies                                  

  The `AWSCodeCommitReadOnlyAccess` managed policy includes the following                                     statements to allow read-only access to notifications. Users with this managed policy                                     applied can view notifications for resources, but cannot create, manage, or subscribe                                     to                                     them.                                   

  ```
  
     {
          "Sid": "CodeStarNotificationsPowerUserAccess",
          "Effect": "Allow",
          "Action": [
              "codestar-notifications:DescribeNotificationRule"
          ],
          "Resource": "*",
          "Condition" : {
              "StringLike" : {"codestar-notifications:NotificationsForResource" : "arn:aws:codecommit:*"} 
          }
      },    
      {
          "Sid": "CodeStarNotificationsListAccess",
          "Effect": "Allow",
          "Action": [
              "codestar-notifications:ListNotificationRules",
              "codestar-notifications:ListEventTypes",
              "codestar-notifications:ListTargets"
          ],
          "Resource": "*"
      }
  ```

  #### Permissions related to notifications in other managed                                     policies                                  

  The `AWSCodeCommitPowerUser` managed policy includes the following statements                                     to allow users to create, edit, and subscribe to notifications. Users cannot delete                                     notification rules or manage tags for resources.                                  

  ```
  
      {
          "Sid": "CodeStarNotificationsReadWriteAccess",
          "Effect": "Allow",
          "Action": [
              "codestar-notifications:CreateNotificationRule",
              "codestar-notifications:DescribeNotificationRule",
              "codestar-notifications:UpdateNotificationRule",
              "codestar-notifications:DeleteNotificationRule",
              "codestar-notifications:Subscribe",
              "codestar-notifications:Unsubscribe"
          ],
          "Resource": "*",
          "Condition" : {
              "StringLike" : {"codestar-notifications:NotificationsForResource" : "arn:aws:codecommit*"} 
          }
      },    
      {
          "Sid": "CodeStarNotificationsListAccess",
          "Effect": "Allow",
          "Action": [
              "codestar-notifications:ListNotificationRules",
              "codestar-notifications:ListTargets",
              "codestar-notifications:ListTagsforResource",
              "codestar-notifications:ListEventTypes"
          ],
          "Resource": "*"
      },
      {
          "Sid": "SNSTopicListAccess",
          "Effect": "Allow",
          "Action": [
              "sns:ListTopics"
          ],
          "Resource": "*"
      },
      {
          "Sid": "CodeStarNotificationsChatbotAccess",
          "Effect": "Allow",
          "Action": [
              "chatbot:DescribeSlackChannelConfigurations"
            ],
         "Resource": "*"
      }
  ```

  For more information about IAM and notifications, see [Identity and Access Management for AWS CodeStar Notifications](https://docs.aws.amazon.com/codestar-notifications/latest/userguide/security-iam.html).                                  

  ### AWS CodeCommit 托管策略和 Amazon CodeGuru Reviewer

  CodeCommit 支持 Amazon CodeGuru Reviewer，后者是一项自动代码审查服务，它使用程序分析和机器学习来检测 Java 或 Python                                     代码中的常见问题并推荐修复方法。CodeCommit 托管策略包含 CodeGuru Reviewer 功能的策略语句。有关更多信息，请参阅[什么是 Amazon CodeGuru Reviewer](https://docs.aws.amazon.com/codeguru/latest/reviewer-ug/welcome.html)。                                  

  #### 中与 CodeGuru Reviewer 相关的权限AWSCodeCommitFullAccess

  `AWSCodeCommitFullAccess` 托管策略包含以下语句，以允许将 CodeGuru Reviewer 与 CodeCommit 存储库关联和取消关联。已应用此托管策略的用户还可以查看 CodeCommit                                     存储库和 CodeGuru Reviewer 之间的关联状态并查看拉取请求的审核作业的状态。                                  

  ```
  
      {
        "Sid": "AmazonCodeGuruReviewerFullAccess",
        "Effect": "Allow",
        "Action": [
          "codeguru-reviewer:AssociateRepository",
          "codeguru-reviewer:DescribeRepositoryAssociation",
          "codeguru-reviewer:ListRepositoryAssociations",
          "codeguru-reviewer:DisassociateRepository",
          "codeguru-reviewer:DescribeCodeReview",
          "codeguru-reviewer:ListCodeReviews"
        ],
        "Resource": "*"
      },
      {
        "Sid": "AmazonCodeGuruReviewerSLRCreation",
        "Action": "iam:CreateServiceLinkedRole",
        "Effect": "Allow",
        "Resource": "arn:aws:iam::*:role/aws-service-role/codeguru-reviewer.amazonaws.com/AWSServiceRoleForAmazonCodeGuruReviewer",
        "Condition": {
          "StringLike": {
            "iam:AWSServiceName": "codeguru-reviewer.amazonaws.com"
          }
        }
      },
      {
        "Sid": "CloudWatchEventsManagedRules",
        "Effect": "Allow",
        "Action": [
          "events:PutRule",
          "events:PutTargets",
          "events:DeleteRule",
          "events:RemoveTargets"
        ],
        "Resource": "*",
        "Condition": {
          "StringEquals": {
            "events:ManagedBy": "codeguru-reviewer.amazonaws.com"
          }
        }
      }
  ```

  #### 中与 CodeGuru Reviewer 相关的权限AWSCodeCommitPowerUser

  托管策略包含以下语句，以允许用户将存储库与 `AWSCodeCommitPowerUser` 关联和取消关联、查看关联状态以及查看拉取请求的审核作业的状态。CodeGuru Reviewer                                  

  ```
  
      {
        "Sid": "AmazonCodeGuruReviewerFullAccess",
        "Effect": "Allow",
        "Action": [
          "codeguru-reviewer:AssociateRepository",
          "codeguru-reviewer:DescribeRepositoryAssociation",
          "codeguru-reviewer:ListRepositoryAssociations",
          "codeguru-reviewer:DisassociateRepository",
          "codeguru-reviewer:DescribeCodeReview",
          "codeguru-reviewer:ListCodeReviews"
        ],
        "Resource": "*"
      },
      {
        "Sid": "AmazonCodeGuruReviewerSLRCreation",
        "Action": "iam:CreateServiceLinkedRole",
        "Effect": "Allow",
        "Resource": "arn:aws:iam::*:role/aws-service-role/codeguru-reviewer.amazonaws.com/AWSServiceRoleForAmazonCodeGuruReviewer",
        "Condition": {
          "StringLike": {
            "iam:AWSServiceName": "codeguru-reviewer.amazonaws.com"
          }
        }
      },
      {
        "Sid": "CloudWatchEventsManagedRules",
        "Effect": "Allow",
        "Action": [
          "events:PutRule",
          "events:PutTargets",
          "events:DeleteRule",
          "events:RemoveTargets"
        ],
        "Resource": "*",
        "Condition": {
          "StringEquals": {
            "events:ManagedBy": "codeguru-reviewer.amazonaws.com"
          }
        }
      }
  ```

  #### 中与 CodeGuru Reviewer 相关的权限AWSCodeCommitReadOnly

  托管策略包含以下语句，以允许对 `AWSCodeCommitReadOnlyAccess` 关联状态进行只读访问并查看拉取请求的审核作业的状态。CodeGuru Reviewer应用了此托管策略的用户无法关联或取消关联存储库。                                  

  ```
  
       {
        "Sid": "AmazonCodeGuruReviewerReadOnlyAccess",
        "Effect": "Allow",
        "Action": [
              "codeguru-reviewer:DescribeRepositoryAssociation",
              "codeguru-reviewer:ListRepositoryAssociations",
              "codeguru-reviewer:DescribeCodeReview",
              "codeguru-reviewer:ListCodeReviews"
        ],
        "Resource": "*"
      }
  ```

  #### Amazon CodeGuru Reviewer 服务相关角色

  当您将存储库与 CodeGuru Reviewer 关联时，将创建一个服务相关角色，以便 CodeGuru Reviewer 可以检测拉取请求中 Java 或 Python                                     代码的问题并推荐修复方法。服务相关角色命名为 AWSServiceRoleForAmazonCodeGuruReviewer。有关更多信息，请参阅[为 Amazon CodeGuru Reviewer 使用服务相关角色](https://docs.aws.amazon.com/codeguru/latest/reviewer-ug/using-service-linked-roles.html)。                                  

  有关更多信息，请参阅 https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_managed-vs-inline.html#aws-managed-policies *中的 IAM 用户指南AWS 托管策略*。                                  

  ## 客户管理的策略示例

  您可以创建自己的自定义 IAM 策略，以授予 CodeCommit 操作和资源的相关权限。您可以将这些自定义策略附加到需要这些权限的 IAM 用户或组。您还可以创建自己的自定义                                     IAM 策略以便集成 CodeCommit 和其他 AWS 服务。                                  

  **主题**

  - [客户托管的身份策略示例](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#customer-managed-policies-identity)
  - [客户托管的集成策略示例](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#integration-policy-examples)

  ### 客户托管的身份策略示例

  以下示例 IAM 策略授予执行各种 CodeCommit 操作的权限。可以使用它们限制 IAM 用户和角色的 CodeCommit 访问。这些策略控制使用 CodeCommit                                     控制台、API、AWS SDKs 或 AWS CLI 执行操作的能力。                                  

  

  ​                                        

  注意

  所有示例都使用 美国西部（俄勒冈）区域 （us-west-2） 并包含虚构账户 IDs。

   **示例**

  - [示例 1：允许用户在单个 AWS 区域中执行 CodeCommit 操作](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#identity-based-policies-example-1)
  - [示例 2：允许用户对单个存储库使用 Git](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#identity-based-policies-example-2)
  - [示例 3：允许从指定 IP 地址范围连接的用户访问存储库 ](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#identity-based-policies-example-3)
  - [示例 4：拒绝或允许对分支执行操作](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#identity-based-policies-example-4)
  - [示例 5：拒绝或允许对具有标签的存储库执行操作](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#identity-based-policies-example-5)

  #### 示例 1：允许用户在单个 AWS 区域中执行 CodeCommit 操作

  以下权限策略使用通配符 (`"codecommit:*"`) 以允许用户在 us-east-2 区域中（而不是在其他 AWS 区域中）执行所有 CodeCommit 操作。                                  

  ```
  
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": "codecommit:*",
              "Resource": "arn:aws:codecommit:us-east-2:111111111111:*",
              "Condition": {
                  "StringEquals": {
                      "aws:RequestedRegion": "us-east-2"
                  }
              }
          },
          {
              "Effect": "Allow",
              "Action": "codecommit:ListRepositories",
              "Resource": "*",
              "Condition": {
                  "StringEquals": {
                      "aws:RequestedRegion": "us-east-2"
                  }
              }
          }
      ]
  }
  ```

  #### 示例 2：允许用户对单个存储库使用 Git

  在 CodeCommit 中，`GitPull` IAM 策略权限适用于从 CodeCommit 检索数据的任何 Git 客户端命令，包括 **git fetch**、**git clone** 等。同样，`GitPush` IAM 策略权限适用于将数据发送到 CodeCommit 的任何 Git 客户端命令。例如，如果 `GitPush` IAM 策略权限设置为 `Allow`，则用户可以使用 Git 协议推送分支删除。对该 IAM 用户的 `DeleteBranch` 操作应用的任何权限都不会影响推送。权限适用于使用控制台、`DeleteBranch`、AWS CLI 和 API 执行的操作，但不适用于通过 Git 协议执行的操作。SDKs                                  

  下面的示例允许指定用户对名为 `MyDemoRepo` 的 CodeCommit 存储库执行提取和推送操作：                                  

  ```
  
  {
    "Version": "2012-10-17",
    "Statement" : [
      {
        "Effect" : "Allow",
        "Action" : [
          "codecommit:GitPull",
          "codecommit:GitPush"
        ],
        "Resource" : "arn:aws:codecommit:us-east-2:111111111111:MyDemoRepo"
      }
    ]
  }
  ```

  #### 示例 3：允许从指定 IP 地址范围连接的用户访问存储库 

  您可以创建策略来只允许其 IP 地址位于特定 IP 地址范围内的用户连接 CodeCommit 存储库。可通过两种等效方法来实现此目的。一种是创建 `Deny` 策略，当用户 IP 地址不在特定块内时禁止 CodeCommit 操作；另一种是创建 `Allow` 策略，当用户 IP 地址在特定块内时允许 CodeCommit 操作。                                  

  您可以创建 `Deny` 策略，拒绝在特定 IP 范围之外的所有用户的访问。例如，您可以向需要访问存储库的所有用户附加 AWSCodeCommitPowerUser 托管策略和客户托管策略。下面的示例策略拒绝其                                     IP 地址不在指定 IP 地址块 203.0.113.0/16 内的用户的所有 CodeCommit 权限：                                  

  ```
  
  {
     "Version": "2012-10-17",
     "Statement": [
        {
           "Effect": "Deny",
           "Action": [
              "codecommit:*"
           ],
           "Resource": "*",
           "Condition": {
              "NotIpAddress": {
                 "aws:SourceIp": [
                    "203.0.113.0/16"
                 ]
              }
           }
        }
     ]
  }
  ```

  下面的示例策略允许具有 AWSCodeCommitPowerUser 托管策略的等效权限的指定用户在其 IP 地址位于指定的地址块 203.0.113.0/16 内时访问名为                                     MyDemoRepo 的 CodeCommit 存储库：                                  

  ```
  
  {
     "Version": "2012-10-17",
     "Statement": [
        {
           "Effect": "Allow",
           "Action": [
              "codecommit:BatchGetRepositories",
              "codecommit:CreateBranch",
              "codecommit:CreateRepository",
              "codecommit:Get*",
              "codecommit:GitPull",
              "codecommit:GitPush",
              "codecommit:List*",
              "codecommit:Put*",
              "codecommit:Post*",
              "codecommit:Merge*",
              "codecommit:TagResource",
              "codecommit:Test*",
              "codecommit:UntagResource",
              "codecommit:Update*"
           ],
           "Resource": "arn:aws:codecommit:us-east-2:111111111111:MyDemoRepo",
           "Condition": {
              "IpAddress": {
                 "aws:SourceIp": [
                    "203.0.113.0/16"
                 ]
              }
           }
        }
     ]
  }
  ```

  

  #### 示例 4：拒绝或允许对分支执行操作

  您可以创建一条策略，拒绝用户在一个或多个分支上执行指定操作的权限。或者，您可以创建一条策略，允许在一个或多个分支上执行某些操作，但在该存储库的其他分支上则不允许执行这些操作。这些策略可与相应的托管                                     (预定义) 策略结合使用。有关更多信息，请参阅[限制推送和合并到分支 AWS CodeCommit](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-conditional-branch.html)。                                  

  例如，您可以创建一个 `Deny` 策略，拒绝用户对名为 的存储库中名为 master 的分支进行更改（包括删除该分支），`MyDemoRepo`， 您可以将此策略与 **AWSCodeCommitPowerUser** 托管策略结合使用。已应用这两个策略的用户将能够创建和删除分支、创建拉取请求以及执行 **AWSCodeCommitPowerUser** 允许的所有其他操作，但无法将更改推送到名为 *master* 的分支、在  *控制台的* masterCodeCommit 分支中添加或删除文件，或将分支或拉取请求合并到 *master* 分支中。由于 `Deny` 应用于 `GitPush`，您必须在该策略中包含 `Null` 语句，当用户从本地存储库进行推送时，分析初始 `GitPush` 调用是否有效。                                  

  ​                                        

  提示

  如果您希望创建一条策略，应用于您的  *账户的所有存储库中名为* masterAWS 的所有分支，对于 `Resource`，请指定星号 ( `*` ) 而不是存储库 ARN。                                        

  ```
  
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Deny",
              "Action": [
                  "codecommit:GitPush",
                  "codecommit:DeleteBranch",
                  "codecommit:PutFile",
                  "codecommit:Merge*"
              ],
              "Resource": "arn:aws:codecommit:us-east-2:111111111111:MyDemoRepo",
              "Condition": {
                  "StringEqualsIfExists": {
                      "codecommit:References": [
                          "refs/heads/master"   
                      ]
                  },
                  "Null": {
                      "codecommit:References": false
                  }
              }
          }
      ]
  }
  ```

  以下示例策略允许用户对 AWS 账户的所有存储库中名为 master 的分支进行更改。它不允许更改任何其他分支。可以将此策略与 AWSCodeCommitReadOnly                                     托管策略结合使用，以允许自动推送到主分支中的存储库。由于效果为 `Allow`，所以此示例策略无法与 AWSCodeCommitPowerUser 这样的托管策略结合使用。                                  

  ```
  
  {
      "Version": "2012-10-17",
      "Statement": [
          {
              "Effect": "Allow",
              "Action": [
                  "codecommit:GitPush",
                  "codecommit:Merge*"
              ],
              "Resource": "*",
              "Condition": {
                  "StringEqualsIfExists": {
                      "codecommit:References": [
                          "refs/heads/master"
                      ]
                  }
              }
          }
      ]
  }
  ```

  

  #### 示例 5：拒绝或允许对具有标签的存储库执行操作

  您可以创建一个使用与存储库关联的 AWS 标签来允许或拒绝对这些存储库执行操作的策略，然后将该策略应用于为管理 IAM 用户而配置的 IAM 组。例如，您可以创建一个策略，拒绝对具有                                     CodeCommit 标签键 AWSStatus *和键值* Secret *的任何存储库执行所有*  操作，然后将该策略应用于为常规开发人员创建的 IAM 组 （`Developers`）。 然后，您需要确保使用这些已标记存储库的开发人员不是该常规 的成员 `Developers` 组，但属于未应用限制性策略的其他 IAM 组 （*SecretDevelopers*）。                                  

  以下示例拒绝对使用键 CodeCommitStatus *和键值* Secret *标记的存储库执行所有*  操作：                                  

  ```
  
  {
    "Version": "2012-10-17",
    "Statement" : [
      {
        "Effect" : "Deny",
        "Action" : "codecommit:*"
        "Resource" : "*",
        "Condition" : {
           "StringEquals" : "aws:ResourceTag/Status": "Secret"
          }
      }
    ]
  }
  ```

  您可以通过将特定存储库而不是所有存储库指定为资源来进一步优化此策略。您还可以创建策略以允许对未使用特定标签标记的所有存储库执行 CodeCommit 操作。例如，以下策略为除了使用指定标签标记的存储库以外的所有其他存储库提供与                                     AWSCodeCommitPowerUser 等效的权限：                                  

  ```
  
  {
     "Version": "2012-10-17",
     "Statement": [
        {
           "Effect": "Allow",
           "Action": [
              "codecommit:BatchGetRepositories",
              "codecommit:CreateBranch",
              "codecommit:CreateRepository",
              "codecommit:Get*",
              "codecommit:GitPull",
              "codecommit:GitPush",
              "codecommit:List*",
              "codecommit:Put*",
              "codecommit:TagResource",
              "codecommit:Test*",
              "codecommit:UntagResource",
              "codecommit:Update*"
           ],
           "Resource": "*",
           "Condition": {
              "StringNotEquals": {
                 "aws:ResourceTag/Status": "Secret",
                 "aws:ResourceTag/Team": "Saanvi"
              }
           }
        }
     ]
  }
  ```

  ### 客户托管的集成策略示例

  本节提供的示例客户托管用户策略授予在 CodeCommit 和其他 AWS 服务之间进行集成的权限。有关允许对 CodeCommit 存储库进行跨账户访问的特定策略示例，请参阅[配置跨帐户访问 AWS CodeCommit 使用角色的存储库](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/cross-account.html)。                                  

  ​                                        

  注意

  需要 AWS 区域时，所有示例都使用 美国西部（俄勒冈）区域 （us-west-2），并且包含虚构的账户 IDs。

   **示例**

  - [示例 1：创建允许对 Amazon SNS 主题进行跨账户访问的策略](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#access-permissions-sns-int)
  - [示例 2：创建 Amazon Simple Notification Service （Amazon SNS） 主题策略以允许 Amazon CloudWatch                                                 Events 将 CodeCommit 事件发布到该主题 ](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#access-permissions-SNS-CWE)
  - [示例 3：为 AWS Lambda 与 CodeCommit 触发器的集成创建策略](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/auth-and-access-control-iam-identity-based-access-control.html#access-permissions-lambda-int)

  #### 示例 1：创建允许对 Amazon SNS 主题进行跨账户访问的策略

  您可以对 CodeCommit  存储库进行配置，以使代码推送或其他事件能够触发操作，例如从 Amazon Simple Notification Service (Amazon                                     SNS) 发送通知。如果使用创建 CodeCommit 存储库的账户创建  Amazon SNS 主题，则无需配置其他 IAM 策略或权限。您可以创建主题，然后为存储库创建触发器。有关更多信息，请参阅[为 Amazon SNS 主题创建触发器](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-notify-sns.html)。                                  

  但是，如果要将触发器配置为使用其他 AWS 账户中的 Amazon SNS 主题，则必须先为该主题配置允许 CodeCommit 向该主题发布内容的策略。从其他账户打开                                     Amazon SNS 控制台，从列表中选择该主题，对于 **Other topic actions (其他主题操作)**，选择 **Edit topic policy (编辑主题策略)**。在 **Advanced (高级)** 选项卡上，修改该主题的策略，以允许 CodeCommit 向该主题发布内容。例如，如果策略是默认策略，则您需要按如下所示修改策略，更改 中的项目 `red italic text` ，以匹配您的存储库、Amazon SNS 主题和账户的值：                                  

  ```
  
  {
    "Version": "2008-10-17",
    "Id": "__default_policy_ID",
    "Statement": [
      {
        "Sid": "__default_statement_ID",
        "Effect": "Allow",
        "Principal": {
          "AWS": "*"
        },
        "Action": [
          "sns:Subscribe",
          "sns:ListSubscriptionsByTopic",
          "sns:DeleteTopic",
          "sns:GetTopicAttributes",
          "sns:Publish",
          "sns:RemovePermission",
          "sns:AddPermission",
          "sns:Receive",
          "sns:SetTopicAttributes"
        ],
        "Resource": "arn:aws:sns:us-east-2:111111111111:NotMySNSTopic",
        "Condition": {
          "StringEquals": {
            "AWS:SourceOwner": "111111111111"
          }
        }
       },
       {
        "Sid": "CodeCommit-Policy_ID",
        "Effect": "Allow",
        "Principal": {
          "Service": "codecommit.amazonaws.com"
        },
        "Action": "sns:Publish",
        "Resource": "arn:aws:sns:us-east-2:111111111111:NotMySNSTopic",
        "Condition": {
          "StringEquals": {
            "AWS:SourceArn": "arn:aws:codecommit:us-east-2:111111111111:MyDemoRepo",
            "AWS:SourceAccount": "111111111111"
          }
        }
      }
    ]
  }
  ```

  #### 示例 2：创建 Amazon Simple Notification Service （Amazon SNS） 主题策略以允许 Amazon CloudWatch                                     Events 将 CodeCommit 事件发布到该主题                                   

  您可以将 CloudWatch Events 配置为在事件（包括 CodeCommit 事件）发生时发布到 Amazon SNS 主题。为此，您必须确保 CloudWatch                                     Events 有权将事件发布到您的 Amazon SNS 主题，方式是通过为主题创建策略或修改主题的现有策略，类似于以下内容：                                  

  ```
  
  {
    Version":"2012-10-17",
    "Id":"__default_policy_ID",
    "Statement":[
      {
        "Sid":"__default_statement_ID",
        "Effect":"Allow",
        "Principal":"{"AWS":"*"},
        "Action":
          "sns:Publish"
        ]
        "Resource":"arn:aws:sns:us-east-2:123456789012:MyTopic",
        "Condition":{
          "StringEquals":{"AWS:SourceOwner":123456789012"}
        }
      },                    
      {
        "Sid":"Allow_Publish_Events",
        "Effect":"Allow",
        "Principal":{"Service":"events.amazonaws.com"},
        "Action":"sns:Publish",
        "Resource":"arn:aws:sns:us-east-2:123456789012:MyTopic"
      }
    ]
  }
  ```

  有关 CodeCommit 和 CloudWatch Events 的更多信息，请参阅[受支持服务的 CloudWatch Events 事件示例](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/EventTypes.html#codecommit_event_type)。有关 IAM 和策略语言的更多信息，请参阅 [IAM JSON 策略语言的语法](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_grammar.html)。                                  

  #### 示例 3：为 AWS Lambda 与 CodeCommit 触发器的集成创建策略

  您可以配置 CodeCommit 存储库以使代码推送或其他事件能够触发操作，例如调用 AWS Lambda 中的函数。有关更多信息，请参阅[为 Lambda 功能](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-notify-lambda.html)。此信息特定于触发器而不是 CloudWatch Events。                                  

  如果您需要让触发器直接运行 Lambda 函数（而不是使用 Amazon SNS 主题调用 Lambda 函数），而且您未在 Lambda 控制台中配置该触发器，则必须在该函数的资源策略中包含类似以下内容的策略：

  ```
  
  {
    "Statement":{
       "StatementId":"Id-1",
       "Action":"lambda:InvokeFunction",
       "Principal":"codecommit.amazonaws.com",
       "SourceArn":"arn:aws:codecommit:us-east-2:111111111111:MyDemoRepo",
       "SourceAccount":"111111111111"
    }
  }
  ```

  在手动配置调用 CodeCommit 函数的 Lambda 触发器时，您还必须使用 Lambda [AddPermission](https://docs.aws.amazon.com/lambda/latest/dg/API_AddPermission.html) 命令授予权限以便 CodeCommit 调用该函数。有关示例，请参阅[为现有 Lambda 功能](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-notify-lambda-cc.html)的[允许 CodeCommit 运行 Lambda 函数](https://docs.aws.amazon.com/zh_cn/codecommit/latest/userguide/how-to-notify-lambda-cc.html#how-to-notify-lambda-create-function-perm)部分。                                  

  有关 Lambda 函数资源策略的更多信息，请参阅 [ 开发人员指南AddPermission 中的](https://docs.aws.amazon.com/lambda/latest/dg/API_AddPermission.html)[和](https://docs.aws.amazon.com/lambda/latest/dg/intro-invocation-modes.html)拉/推事件模型*。AWS Lambda*

- # What is Amazon CodeGuru Reviewer?

   [ PDF   ](https://docs.aws.amazon.com/zh_cn/codeguru/latest/reviewer-ug/reviewer-ug.pdf#welcome)    

   [ RSS   ](https://docs.aws.amazon.com/zh_cn/codeguru/latest/reviewer-ug/service-guide.rss)    

   

  Amazon CodeGuru Reviewer is a service that uses program analysis and machine learning                                     to detect potential                                     		defects that are difficult for developers to find and offers suggestions for improving                                     your Java                                     		and Python code. This service has been released for general availability in several                                     [regions](https://docs.aws.amazon.com/general/latest/gr/codeguru-reviewer.html).                                  

  By proactively detecting code defects, CodeGuru Reviewer can provide guidelines for                                     addressing them and                                     implementing best practices to improve the overall quality and maintainability of                                     your code                                     base during the code review stage. For more information, see [How CodeGuru Reviewer works](https://docs.aws.amazon.com/zh_cn/codeguru/latest/reviewer-ug/how-codeguru-reviewer-works.html).                                  

  ## What kind of recommendations does CodeGuru Reviewer provide?

  CodeGuru Reviewer doesn't flag syntactical mistakes, as these are relatively easy                                     to find. Instead, CodeGuru Reviewer will identify more complex problems and suggest                                     improvements related to the following:                                   

  - AWS best practices
  - Concurrency
  - Resource leak prevention
  - Sensitive information leak prevention
  - Common coding best practices
  - Refactoring
  - Input validation
  - Security analysis
  - Code quality

  For more information, see [Recommendations in Amazon CodeGuru Reviewer](https://docs.aws.amazon.com/zh_cn/codeguru/latest/reviewer-ug/recommendations.html).                                  

  ## What languages and repositories can I use with                                     				CodeGuru Reviewer?                                  

  CodeGuru Reviewer is designed to work with Java and Python code repositories in the                                     following source                                     			providers:                                  

  - [AWS CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/welcome.html)
  - Bitbucket
  - GitHub
  - GitHub Enterprise Cloud
  - GitHub Enterprise Server
  - Amazon S3

   If you use any of these source providers, you can integrate with CodeGuru Reviewer                                     with just a few                                     			steps. After you associate a repository with CodeGuru Reviewer, you can [interact with                                        				recommendations in the CodeGuru Reviewer console](https://docs.aws.amazon.com/codeguru/latest/reviewer-ug/give-feedback-from-code-review-details.html). For pull request code reviews, you can also                                     				[see recommendations directly from inside pull requests](https://docs.aws.amazon.com/codeguru/latest/reviewer-ug/provide-feedback.html#provide-feedback-in-pull-requests) in your repository                                     			context.                                  

- [yunikorn 介绍](https://github.com/apache/incubator-yunikorn-core)