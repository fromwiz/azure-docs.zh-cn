## <a name="create-a-storage-account-by-using-the-azure-portal"></a>使用 Azure 门户创建存储帐户

首先，创建用于本快速入门的新通用存储帐户。 

1. 转到 [Azure 门户](https://portal.azure.com/#create/Microsoft.StorageAccount-ARM)，使用 Azure 帐户登录。 
2. 为存储帐户输入唯一的名称。 请记住以下存储帐户命名规则：
    - 名称长度必须为 3 到 24 个字符。
    - 名称只能包含数字和小写字母。
3. 请确保设置以下默认值： 
    - “部署模型”设置为“资源管理器”。
    - “帐户类型”设置为“常规用途”。
    - “性能”设置为“标准”。
    - “复制”设置为“本地冗余存储(LRS)”。
4. 选择订阅。 
5. 对于“资源组”，创建一个新资源组，并为其赋予唯一名称。 
6. 选择要用于存储帐户的位置。
7. 依次选择“固定到仪表板”、“创建”，创建存储帐户。 

创建存储帐户后，该帐户将固定到仪表板。 选择它以将其打开。 在“设置”下，选择“访问密钥”。 选择主密钥并将关联的连接字符串复制到剪贴板， 然后将该字符串粘贴到文本编辑器，以供将来使用。
