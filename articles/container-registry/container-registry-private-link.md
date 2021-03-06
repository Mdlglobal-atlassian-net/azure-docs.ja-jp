---
title: プライベート リンクを設定する
description: コンテナー レジストリにプライベート エンドポイントを設定し、ローカル仮想ネットワークでプライベート リンクを有効にする
ms.topic: article
ms.date: 03/10/2020
ms.openlocfilehash: de8228d84497e71f24dba3dd4e6162cb6735a8c1
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2020
ms.locfileid: "79498919"
---
# <a name="configure-azure-private-link-for-an-azure-container-registry"></a>Azure コンテナー レジストリ用に Azure Private Link を構成する 

Azure 仮想ネットワーク上のクライアントが[プライベート リンク](../private-link/private-link-overview.md)経由でレジストリに安全にアクセスできるように、Azure コンテナー レジストリの[プライベート エンドポイント](../private-link/private-endpoint-overview.md)を設定することができます。 プライベート エンドポイントでは、レジストリのために仮想ネットワークのアドレス空間からの IP アドレスを使用します。 仮想ネットワーク上のクライアントとレジストリ間のネットワーク トラフィックは、仮想ネットワークおよび Microsoft バックボーン ネットワーク上のプライベート リンクを経由することで、パブリック インターネットにさらされないようにします。

設定がレジストリの割り当てられたプライベート IP アドレスに解決されるように、プライベート エンドポイント用に [DNS 設定を構成する](../private-link/private-endpoint-overview.md#dns-configuration)ことができます。 DNS 構成では、ネットワーク内のクライアントとサービスは、レジストリの完全修飾ドメイン名 (*myregistry.azurecr.io*など) で引き続きレジストリにアクセスできます。

この機能は、**Premium** コンテナー レジストリ サービス レベルで使用できます。 レジストリ サービスのレベルと制限については、「[Azure Container Registry SKU](container-registry-skus.md)」をご覧ください。

> [!IMPORTANT]
> この機能は現在プレビュー段階であり、一定の[制限事項](#preview-limitations)が適用されます。 プレビュー版は、[追加使用条件][terms-of-use]に同意することを条件に使用できます。 この機能の一部の側面は、一般公開 (GA) 前に変更される可能性があります。

## <a name="preview-limitations"></a>プレビューの制限事項

* 現在、[geo レプリケートされたレジストリ](container-registry-geo-replication.md)に、プライベート エンドポイントを持つプライベート リンクを設定することはできません。 

## <a name="prerequisites"></a>前提条件

* この記事の Azure CLI の手順を使用する場合は、Azure CLI バージョン 2.2.0 以降をお勧めします。 インストールまたはアップグレードする必要がある場合は、[Azure CLI のインストール][azure-cli]に関するページを参照してください。 または、[Azure Cloud Shell](../cloud-shell/quickstart.md) で実行します。
* コンテナー レジストリがまだない場合は、1 つ作成し (Premium レベルが必要)、Docker Hub から `hello-world` などのサンプル イメージをプッシュします。 たとえば、[Azure portal][quickstart-portal] または [Azure CLI][quickstart-cli] を使用してレジストリを作成します。
* 別の Azure サブスクリプションのプライベート リンクを使用してレジストリ アクセスを構成する場合、そのサブスクリプションで Azure Container Registry のリソース プロバイダーを登録する必要があります。 次に例を示します。

  ```azurecli
  az account set --subscription <Name or ID of subscription of private link>

  az provider register --namespace Microsoft.ContainerRegistry
  ``` 

この記事の Azure CLI 例では、次の環境変数を使用します。 ご利用の環境に適した値に置き換えてください。 すべての例は、Bash シェル用に書式設定されています。

```bash
registryName=<container-registry-name>
registryLocation=<container-registry-location> # Azure region such as westeurope where registry created
resourceGroup=<resource-group-name>
vmName=<virtual-machine-name>
```

## <a name="create-a-docker-enabled-virtual-machine"></a>Docker 対応仮想マシンの作成

テスト目的の場合は、Docker 対応の Ubuntu VM を使用して、Azure コンテナー レジストリにアクセスします。 レジストリに Azure Active Directory 認証を使用するには、VM に [Azure CLI][azure-cli] もインストールします。 既に Azure 仮想マシンがある場合は、この作成手順を省略します。

仮想マシンとコンテナー レジストリに同じリソース グループを使用できます。 このセットアップでは最終的にクリーンアップが簡素化されますが、必須ではありません。 仮想マシンと仮想ネットワークに別のリソース グループを作成することを選択する場合は、[az group create][az-group-create] を実行します。

```azurecli
az group create --name $resourceGroup --location $registryLocation
```

ここで、[az vm create][az-vm-create] を使用して、既定の Ubuntu Azure 仮想マシンをデプロイします。 次の例では、*myDockerVM* という名前の VM を作成します。

```azurecli
az vm create \
  --resource-group $resourceGroup \
  --name $vmName \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

VM が作成されるまで､数分間かかります｡ コマンドが完了したら、Azure CLI によって表示された `publicIpAddress` をメモします。 このアドレスは、VM への SSH 接続を作成するために使用します。

### <a name="install-docker-on-the-vm"></a>VM に Docker をインストールする

VM が実行されたら、VM への SSH 接続を作成します。 *publicIpAddress* を VM のパブリック IP アドレスに置き換えます。

```bash
ssh azureuser@publicIpAddress
```

次のコマンドを実行して、Ubuntu VM に Docker をインストールします。

```bash
sudo apt-get update
sudo apt install docker.io -y
```

インストールの後、次のコマンドを実行して、VM 上で Docker が正しく実行されていることを確認します。

```bash
sudo docker run -it hello-world
```

出力:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]
```

### <a name="install-the-azure-cli"></a>Azure CLI のインストール

Ubuntu 仮想マシンに Azure CLI をインストールするには、「[apt での Azure CLI のインストール](/cli/azure/install-azure-cli-apt?view=azure-cli-latest)」の手順に従います。 次に例を示します。

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

SSH 接続を終了します。

## <a name="set-up-private-link---cli"></a>プライベート リンクを設定する - CLI

### <a name="get-network-and-subnet-names"></a>ネットワークとサブネットの名前を取得する

まだない場合は、プライベート リンクを設定するための仮想ネットワークとサブネットの名前が必要になります。 この例では、VM とレジストリのプライベート エンドポイントに同じサブネットを使用します。 しかし、多くのシナリオでは、別のサブネットにエンドポイントを設定します。 

VM を作成するときに、既定では Azure によって仮想ネットワークが同じリソース グループに作成されます。 仮想ネットワークの名前は仮想マシンの名前に基づいています。 たとえば、仮想マシンに *myDockerVM* という名前を付ける場合、既定の仮想ネットワーク名は *myDockerVMVNET* で、サブネット名は *myDockerVMSubnet* です。 [az network vnet list][az-network-vnet-list] コマンドを実行して、環境変数にこれらの値を設定します。

```azurecli
networkName=$(az network vnet list \
  --resource-group $resourceGroup \
  --query '[].{Name: name}' --output tsv)

subnetName=$(az network vnet list \
  --resource-group $resourceGroup \
  --query '[].{Subnet: subnets[0].name}' --output tsv)

echo networkName=$networkName
echo subnetName=$subnetName
```

### <a name="disable-network-policies-in-subnet"></a>サブネットでネットワーク ポリシーを無効にする

プライベート エンドポイントのサブネットで、ネットワーク セキュリティ グループなどの[ネットワーク ポリシーを無効にします](../private-link/disable-private-endpoint-network-policy.md)。 [az network vnet subnet update][az-network-vnet-subnet-update] を使用して、サブネット構成を更新します。

```azurecli
az network vnet subnet update \
 --name $subnetName \
 --vnet-name $networkName \
 --resource-group $resourceGroup \
 --disable-private-endpoint-network-policies
```

### <a name="configure-the-private-dns-zone"></a>プライベート DNS ゾーンを構成する

プライベート Azure コンテナー レジストリ ドメイン用のプライベート DNS ゾーンを作成します。 後の手順で、この DNS ゾーン内にレジストリ ドメインの DNS レコードを作成します。

プライベート ゾーンを使用して Azure コンテナー レジストリの既定の DNS 解決をオーバーライドするには、ゾーンに **privatelink.azurecr.io** という名前を付ける必要があります。 次の [az network private-dns zone create][az-network-private-dns-zone-create] コマンドを実行して、プライベート ゾーンを作成します。

```azurecli
az network private-dns zone create \
  --resource-group $resourceGroup \
  --name "privatelink.azurecr.io"
```

### <a name="create-an-association-link"></a>関連付けリンクを作成する

[az network private-dns link vnet create][az-network-private-dns-link-vnet-create] を実行して、プライベート ゾーンを仮想ネットワークに関連付けます。 この例では、*myDNSLink* というリンクを作成します。

```azurecli
az network private-dns link vnet create \
  --resource-group $resourceGroup \
  --zone-name "privatelink.azurecr.io" \
  --name MyDNSLink \
  --virtual-network $networkName \
  --registration-enabled false
```

### <a name="create-a-private-registry-endpoint"></a>プライベート レジストリ エンドポイントを作成する

このセクションでは、仮想ネットワーク内にレジストリのプライベート エンドポイントを作成します。 まず、レジストリのリソース ID を取得します。

```azurecli
registryID=$(az acr show --name $registryName \
  --query 'id' --output tsv)
```

[az network private endpoint create][az-network-private-endpoint-create] コマンドを実行して、レジストリのプライベート エンドポイントを作成します。

次の例では、*myPrivateEndpoint* というエンドポイントと *myConnection* というサービス接続を作成します。 エンドポイントのコンテナー レジストリ リソースを指定するには、`--group-ids registry` を渡します。

```azurecli
az network private-endpoint create \
    --name myPrivateEndpoint \
    --resource-group $resourceGroup \
    --vnet-name $networkName \
    --subnet $subnetName \
    --private-connection-resource-id $registryID \
    --group-ids registry \
    --connection-name myConnection
```

### <a name="get-private-ip-addresses"></a>プライベート IP アドレスを取得する

[az network private endpoint show][az-network-private-endpoint-show] を実行し、ネットワーク インターフェイス ID について、エンドポイントにクエリを実行します。

```azurecli
networkInterfaceID=$(az network private-endpoint show \
  --name myPrivateEndpoint \
  --resource-group $resourceGroup \
  --query 'networkInterfaces[0].id' \
  --output tsv)
```

ネットワーク インターフェイスには、コンテナーレジストリの 2 つのプライベート IP アドレスが関連付けられています。1 つはレジストリ自体用で、もう 1 つはレジストリのデータ エンドポイント用です。 次の [az resource show][az-resource-show] コマンドを実行し、コンテナー レジストリとレジストリのデータ エンドポイント用のプライベート IP アドレスを取得します。

```azurecli
privateIP=$(az resource show \
  --ids $networkInterfaceID \
  --api-version 2019-04-01 --query 'properties.ipConfigurations[1].properties.privateIPAddress' \
  --output tsv)

dataEndpointPrivateIP=$(az resource show \
  --ids $networkInterfaceID \
  --api-version 2019-04-01 \
  --query 'properties.ipConfigurations[0].properties.privateIPAddress' \
  --output tsv)
```

### <a name="create-dns-records-in-the-private-zone"></a>プライベート ゾーンに DNS レコードを作成する

次のコマンドでは、レジストリ エンドポイントとそのデータ エンドポイントのプライベート ゾーンに DNS レコードを作成します。 たとえば、*westeurope* リージョンに *myregistry* という名前のレジストリがある場合、エンドポイント名は `myregistry.azurecr.io` と `myregistry.westeurope.data.azurecr.io` になります。 

まず、[az network private-dns record-set a create][az-network-private-dns-record-set-a-create] を実行し、レジストリ エンドポイントとデータ エンドポイント用の空の A レコード セットを作成します。

```azurecli
az network private-dns record-set a create \
  --name $registryName \
  --zone-name privatelink.azurecr.io \
  --resource-group $resourceGroup

# Specify registry region in data endpoint name
az network private-dns record-set a create \
  --name ${registryName}.${registryLocation}.data \
  --zone-name privatelink.azurecr.io \
  --resource-group $resourceGroup
```

[az network private-dns record-set a add-record][az-network-private-dns-record-set-a-add-record] コマンドを実行し、レジストリ エンドポイントとデータ エンドポイント用の A レコードを作成します。

```azurecli
az network private-dns record-set a add-record \
  --record-set-name $registryName \
  --zone-name privatelink.azurecr.io \
  --resource-group $resourceGroup \
  --ipv4-address $privateIP

# Specify registry region in data endpoint name
az network private-dns record-set a add-record \
  --record-set-name ${registryName}.${registryLocation}.data \
  --zone-name privatelink.azurecr.io \
  --resource-group $resourceGroup \
  --ipv4-address $dataEndpointPrivateIP
```

これで、プライベート リンクが構成され、使用できるようになりました。

## <a name="set-up-private-link---portal"></a>プライベート リンクを設定する - ポータル

次の手順では、仮想ネットワークとサブネットがテスト用の VM で既に設定されていることを前提としています。 [新しい仮想ネットワークとサブネットを作成する](../virtual-network/quick-create-portal.md)こともできます。

### <a name="create-a-private-endpoint"></a>プライベート エンドポイントの作成

1. ポータルで、自分のコンテナー レジストリに移動します。
1. **[設定]** で、 **[プライベート エンドポイント接続 (プレビュー)]** を選択します。
1. **[+ プライベート エンドポイント]** を選択します。
1. **[基本]** タブで、次の情報を入力または選択します。

    | 設定 | 値 |
    | ------- | ----- |
    | **プロジェクトの詳細** | |
    | サブスクリプション | サブスクリプションを選択します。 |
    | Resource group | 既存のグループの名前を入力するか、新しいものを作成します。|
    | **インスタンスの詳細** |  |
    | 名前 | 一意の名前を入力します。 |
    |リージョン|リージョンを選択します。|
    |||
5. **リソース** を選択します。
6. 次の情報を入力または選択します。

    | 設定 | 値 |
    | ------- | ----- |
    |接続方法  | **[マイ ディレクトリ内の Azure リソースに接続します]** を選択します。|
    | サブスクリプション| サブスクリプションを選択します。 |
    | リソースの種類 | **[Microsoft.ContainerRegistry/registries]** を選択します。 |
    | リソース |レジストリの名前を選択します|
    |ターゲット サブリソース |**レジストリ**を選択します|
    |||
7. **構成** を選択します。
8. 次の情報を入力または選択します。

    | 設定 | 値 |
    | ------- | ----- |
    |**ネットワーク**| |
    | 仮想ネットワーク| *myDockerVMVNET* など、仮想マシンがデプロイされている仮想ネットワークを選択します。 |
    | Subnet | 仮想マシンがデプロイされている *myDockerVMSubnet* などのサブネットを選択します。 |
    |**プライベート DNS の統合**||
    |プライベート DNS ゾーンとの統合 |**[はい]** を選択します。 |
    |プライベート DNS ゾーン |*(新規) privatelink.azurecr.io* を選択します |
    |||

1. **[Review + create]\(レビュー + 作成\)** を選択します。 **[確認および作成]** ページが表示され、Azure によって構成が検証されます。 
2. "**証に成功しました**" というメッセージが表示されたら、 **[作成]** を選択します。

プライベート エンドポイントが作成された後、エンドポイントの **[概要]** ページにプライベート ゾーンの DNS 設定が表示されます。

![エンドポイントの DNS 設定](./media/container-registry-private-link/private-endpoint-overview.png)

これで、プライベート リンクが構成され、使用できるようになりました。

## <a name="validate-private-link-connection"></a>プライベート リンク接続を確認する

プライベート エンドポイントのサブネット内のリソースがプライベート IP アドレスを使用してレジストリに接続され、適切なプライベート DNS ゾーンの統合が行われていることを確認する必要があります。

プライベート リンク接続を確認するには、仮想ネットワークに設定した仮想マシンに SSH 接続します。

`nslookup` コマンドを実行し、プライベート リンク経由のレジストリの IP アドレスを解決します。

```bash
nslookup $registryName.azurecr.io
```

出力例では、サブネットのアドレス空間にあるレジストリの IP アドレスが示されています。

```console
[...]
myregistry.azurecr.io       canonical name = myregistry.privatelink.azurecr.io.
Name:   myregistry.privatelink.azurecr.io
Address: 10.0.0.6
```

この結果と、パブリック エンドポイント経由の同じレジストリの `nslookup` 出力のパブリック IP アドレスを比較します。

```console
[...]
Non-authoritative answer:
Name:   myregistry.westeurope.cloudapp.azure.com
Address: 40.78.103.41
```

### <a name="registry-operations-over-private-link"></a>プライベート リンク経由のレジストリ操作

また、サブネット内の仮想マシンからレジストリ操作を実行できることを確認します。 仮想マシンへの SSH 接続を行い、[az acr login][az-acr-login] を実行してレジストリにログインします。 VM の構成によっては、次のコマンドの前に `sudo` を付けることが必要になる場合があります。

```bash
az acr login --name $registryName
```

`docker pull` などのレジストリ操作を実行して、サンプル イメージをレジストリからプルします。 `hello-world:v1` を、レジストリに適したイメージとタグに置き換え、前にレジストリ ログイン サーバー名 (すべて小文字) を付けます。

```bash
docker pull myregistry.azurecr.io/hello-world:v1
``` 

Docker によってイメージが VM に正常にプルされます。

## <a name="manage-private-endpoint-connections"></a>プライベート エンドポイント接続を管理する

Azure portal を使用するか、[az acr private-endpoint-connection][az-acr-private-endpoint-connection] コマンド グループのコマンドを使って、レジストリのプライベート エンドポイント接続を管理します。 操作には、レジストリのプライベート エンドポイント接続の承認、削除、一覧表示、拒否、または詳細の表示が含まれます。

たとえば、レジストリのプライベート エンドポイント接続の一覧を表示するには、[az acr private-endpoint-connection list][az-acr-private-endpoint-connection-list] コマンドを実行します。 次に例を示します。

```azurecli
az acr private-endpoint-connection list \
  --registry-name $registryName 
```

この記事の手順を使用してプライベート エンドポイント接続を設定すると、レジストリでは、レジストリに対する RBAC アクセス許可を持つクライアントとサービスからの接続が自動的に受け入れられます。 エンドポイントを設定して、手動による接続の承認を要求することができます。 プライベート エンドポイント接続を承認および拒否する方法については、「[プライベート エンドポイント接続を管理する](../private-link/manage-private-endpoint.md)」を参照してください。

## <a name="clean-up-resources"></a>リソースをクリーンアップする

すべての Azure リソースを同じリソース グループ内に作成し、それらが不要になった場合は、1 つの [az group delete](/cli/azure/group) コマンドを使用してリソースを削除することもできます。

```azurecli
az group delete --name $resourceGroup
```

ポータルでリソースをクリーンアップするには、リソース グループに移動します。 リソース グループが読み込まれたら、 **[リソース グループの削除]** をクリックして、リソース グループとそこに格納されているリソースを削除します。

## <a name="next-steps"></a>次のステップ

* Private Link の詳細については、[Azure Private Link](../private-link/private-link-overview.md) に関するドキュメントを参照してください。
* プライベート リンクの代わりに、レジストリ アクセスを制限するネットワーク アクセス規則を設定することもできます。 詳細については、「[Azure 仮想ネットワークまたはファイアウォール規則を使用して Azure コンテナー レジストリへのアクセスを制限する](container-registry-vnet.md)」を参照してください。

<!-- LINKS - external -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-create]: /cli/azure/acr#az-acr-create
[az-acr-show]: /cli/azure/acr#az-acr-show
[az-acr-repository-show]: /cli/azure/acr/repository#az-acr-repository-show
[az-acr-repository-list]: /cli/azure/acr/repository#az-acr-repository-list
[az-acr-login]: /cli/azure/acr#az-acr-login
[az-acr-private-endpoint-connection]: /cli/azure/acr/private-endpoint-connection
[az-acr-private-endpoint-connection-list]: /cli/azure/acr/private-endpoint-connection#az-acr-private-endpoint-connection-list
[az-acr-private-endpoint-connection-approve]: /cli/azure/acr/private-endpoint-connection#az-acr-private-endpoint-connection-approve
[az-group-create]: /cli/azure/group
[az-role-assignment-create]: /cli/azure/role/assignment#az-role-assignment-create
[az-vm-create]: /cli/azure/vm#az-vm-create
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-show
[az-network-vnet-subnet-update]: /cli/azure/network/vnet/subnet/#az-network-vnet-subnet-update
[az-network-vnet-list]: /cli/azure/network/vnet/#az-network-vnet-list
[az-network-private-endpoint-create]: /cli/azure/network/private-endpoint#az-network-private-endpoint-create
[az-network-private-endpoint-show]: /cli/azure/network/private-endpoint#az-network-private-endpoint-show
[az-network-private-dns-zone-create]: /cli/azure/network/private-dns/zone#az-network-private-dns-zone-create
[az-network-private-dns-link-vnet-create]: /cli/azure/network/private-dns/link/vnet#az-network-private-dns-link-vnet-create
[az-network-private-dns-record-set-a-create]: /cli/azure/network/private-dns/record-set/a#az-network-private-dns-record-set-a-create
[az-network-private-dns-record-set-a-add-record]: /cli/azure/network/private-dns/record-set/a#az-network-private-dns-record-set-a-add-record
[az-resource-show]: /cli/azure/resource#az-resource-show
[quickstart-portal]: container-registry-get-started-portal.md
[quickstart-cli]: container-registry-get-started-azure-cli.md
[azure-portal]: https://portal.azure.com
