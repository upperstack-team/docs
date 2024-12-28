# Como autorizar novos usuários a mexer no EKS

Neste tutorial, vamos explicar como permitir que novos usuários acessem e gerenciem um cluster **EKS** (Elastic Kubernetes Service) na AWS. Para isso, utilizaremos o **IAM** (Identity and Access Management) para conceder permissões e configurar o **Kubeconfig** para autenticação com o cluster EKS.

## Passo 1: Criar Usuário no IAM

1. **Acesse o console AWS**:
   - Vá para o [console do IAM](https://console.aws.amazon.com/iam/).
   
2. **Criar um novo usuário**:
   - Clique em **Users** (Usuários) no menu lateral e depois em **Add user** (Adicionar usuário).
   - Escolha o tipo de acesso: selecione **Programmatic access** (Acesso programático) e **AWS Management Console access** (Acesso ao Console de Gerenciamento da AWS).
   - Para o acesso ao console, você pode escolher uma senha gerada automaticamente ou criar uma senha personalizada.

3. **Atribuir permissões**:
   - Selecione **Attach policies directly** (Anexar políticas diretamente).
   - Adicione a política **AmazonEKSClusterPolicy** e **AmazonEKSWorkerNodePolicy**. Essas políticas fornecem permissões necessárias para interagir com o EKS e os nós de trabalho.

4. **Revisar e criar o usuário**:
   - Revise as configurações e clique em **Create user** (Criar usuário).
   - Anote a chave de acesso e a chave secreta, pois você precisará delas mais tarde.

## Passo 2: Conceder Permissões ao Cluster EKS

Para que o novo usuário possa acessar o cluster EKS, você deve associar o usuário do IAM com o **ConfigMap** do Kubernetes, que gerencia as permissões de acesso.

1. **Editar o ConfigMap**:
   O **ConfigMap** `aws-auth` controla o acesso de usuários e grupos IAM ao cluster EKS.

   Utilize o comando abaixo para editar o **ConfigMap**:

   ```bash
   kubectl edit -n kube-system configmap/aws-auth
   ```

2. **Adicionar o usuário IAM**:
   No editor, adicione o ARN do novo usuário IAM sob `mapUsers`. Isso concederá permissões de acesso ao cluster EKS.

   Exemplo:

   ```yaml
   apiVersion: v1
   kind: ConfigMap
   metadata:
     name: aws-auth
     namespace: kube-system
   data:
     mapRoles: |
       - rolearn: arn:aws:iam::123456789012:role/EKS-Cluster-Admin-Role
         username: eks-admin
         groups:
           - system:masters
     mapUsers: |
       - userarn: arn:aws:iam::123456789012:user/new-eks-user
         username: new-eks-user
         groups:
           - system:masters
   ```

   **Nota**: O grupo `system:masters` concede permissões administrativas. Se você quiser conceder permissões mais restritas, substitua por grupos apropriados (exemplo: `system:basic-user`).

3. **Salvar e aplicar**:
   Após editar, salve e feche o editor. O Kubernetes irá automaticamente aplicar as alterações.

## Passo 3: Configurar o `kubeconfig` para o Novo Usuário

Agora, o usuário precisa configurar o `kubeconfig` para acessar o cluster EKS localmente.

1. **Instalar a AWS CLI**:
   Se o usuário não tiver a AWS CLI instalada, execute:

   ```bash
   pip install awscli
   ```

2. **Configurar as credenciais do AWS CLI**:
   O usuário precisa configurar as credenciais da AWS para que a CLI saiba como autenticar. Execute o seguinte comando e forneça as credenciais do novo usuário (Access Key ID e Secret Access Key):

   ```bash
   aws configure
   ```

   Forneça as seguintes informações:
   - **AWS Access Key ID**: A chave de acesso do novo usuário.
   - **AWS Secret Access Key**: A chave secreta do novo usuário.
   - **Default region name**: A região onde seu cluster EKS está localizado (ex: `us-west-2`).
   - **Default output format**: O formato de saída, como `json`.

3. **Atualizar o kubeconfig**:
   O usuário deve atualizar o arquivo `kubeconfig` para acessar o cluster EKS com o seguinte comando:

   ```bash
   aws eks --region <region> update-kubeconfig --name <cluster-name>
   ```

   Substitua `<region>` pela região do cluster EKS (por exemplo, `us-west-2`) e `<cluster-name>` pelo nome do seu cluster EKS.

4. **Testar a conexão**:
   Após configurar o `kubeconfig`, o usuário pode testar a conexão com o cluster EKS executando:

   ```bash
   kubectl get nodes
   ```

   Se tudo estiver configurado corretamente, o comando retornará a lista de nós do cluster.

## Passo 4: Conceder Permissões Adicionais (Se necessário)

Se você precisar de permissões mais granulares, como permitir que o usuário execute certas ações (como criar pods, serviços, etc.), você pode criar políticas IAM personalizadas e associá-las ao usuário ou grupos IAM.

### Exemplos de políticas de acesso (opcionais):
- **Criar e Gerenciar Pods**: Conceder permissão para criar e gerenciar pods no cluster.
- **Gerenciar ConfigMaps**: Conceder permissão para editar e criar ConfigMaps.

Essas permissões podem ser configuradas no IAM ou diretamente no Kubernetes, criando RBACs (Role-Based Access Control) mais detalhadas.

---

## Conclusão

Agora você configurou o acesso de novos usuários ao cluster **EKS** da AWS. Seguindo esses passos, o usuário poderá interagir com o Kubernetes, desde que tenha as permissões adequadas configuradas tanto no **IAM** quanto no **Kubernetes**.

Caso o usuário precise de permissões mais específicas ou restritas, é possível configurar roles personalizadas no Kubernetes e no IAM para melhor controle.

Se precisar de mais detalhes ou ajuda, é só chamar!
