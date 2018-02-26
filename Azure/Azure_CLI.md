
```
RESOURCE_GROUP=TutorialResources
LOCATION=eastus
VM_NAME=TutorialVM1
ADMIN_USERNAME=admin
IMAGE=UbuntuLTS

az login
az vm create --resource-group ${RESOURCE_GROUP} --name ${VM_NAME} --admin-username ${ADMIN_USERNAME} --image ${UbuntIMAGE} --generate-ssh-keys --verbose
az logout

### キャプチャ
az vm deallocate -g ${RESOURCE_GROUP} -n ${VM_NAME}
az vm generalize -g ${RESOURCE_GROUP} -n ${VM_NAME}
az vm capture -g ${RESOURCE_GROUP} -n ${VM_NAME} --vhd-name-prefix "vhd_${VM_NAME}"
```

### 参考リンク

* Azure Document
  * [完全に構成された仮想マシンの作成](https://docs.microsoft.com/ja-jp/azure/virtual-machines/scripts/virtual-machines-linux-cli-sample-create-vm)
  * [Azure CLI 2.0](https://docs.microsoft.com/en-us/cli/azure/overview?view=azure-cli-latest)
