# kubernetes
### Uma abordagem teórica


*O que é o Kubernetes?*

Explicando por um nível mais alto, é uma ferramenta open source utilizada para 
gerenciar aplicações em *containers*.

Fazendo uma analogia com um serviço de entrega, nossos passos para
enviar uma encomenda seriam: 
1. embalar de acordo com uma especificação
2. colocar algumas informações em uma etiqueta 
3. entregar para a empresa de entregas fazer o resto. 

No *Kubernetes* iríamos "embalar" nossa aplicação em containers, 
criar um manifesto declarativo com todas as informações sobre aquela 
aplicação e entregar para o *kubernetes* fazer todo o trabalho difícil, 
como: em qual nó rodar a aplicação, como baixar imagens e iniciar *containers*,
entre outras coisas.

### Arquitetura

Um *Cluster* é formado pelo Control Plane e um conjunto de nós.
O *Control Plane* é como se fosse o cérebro do *cluster*. Nessa camada que são tomadas
decisões como agendamento de *pods*, monitoramento do *cluster* e respostas aos eventos.

<img src="https://user-images.githubusercontent.com/9805258/124046641-cd9a1780-d9e8-11eb-8c69-ccf02570bb7c.png" width="50%" height="50%">


### Nodes

Nós são formados basicamente por:

<img src="https://user-images.githubusercontent.com/9805258/124044852-b9ecb200-d9e4-11eb-933f-448b700cbedf.png" width="25%" height="25%">


* *Kubelet*

Ele registra o nó no *cluster* e adiciona os requisitos necessários como CPU, RAM e outros recursos.
Aguarda instruções da *API Server*, que envia os *pods*. O *Kubelet* executa as *Pods* e reporta tudo para o *Control Plane*.

* *Container Runtime*

*Kubelets* rodam *pods*, *pods* são um ou mais *containers*.
Mas efetivamente, quem roda os *containers* são as *Containers Runtime*, que sabem como baixar as imagens e buildar e iniciar os containers.
O *Kubernetes* suporta diversos agentes de execução de contêineres: Docker, containerd, CRI-O, e qualquer implementação do Kubernetes CRI (Container Runtime Interface).

* *Kube-proxy*

O *Kube-proxy* é um *proxy* de rede do nó. Ele se certifica de que cada pod tenha seu IP único. 

### Pods

É o menor objeto dentro do *cluster*. *Pods* são um conjunto de *containers* em execução.
Eles tem um ambiente de execução compartilhado. Ou seja, se existir mais de um *container* dentro do *pod*, 
eles terão mesmo IP mas será necessário mapear portas diferentes.

É importante enfatizar que a unidade de escala no *kubernetes* são os *Pods*.
Se você quer escalar a sua aplicação, deve fazer isso adicionando novos *pods* do mesmo *container*.
**Nunca se deve aumentar a escalabilidade aumentando *containers* dentro de um *pod***, apenas mais *pods* de um mesmo *container*.
E o mesmo serve para remover escalabilidade, para isso você deve remover *pods*.

![image](https://user-images.githubusercontent.com/9805258/124047357-6bdaad00-d9ea-11eb-92e1-e33344f13832.png)


### Declarativo vs Imperativo

O *Kubernetes* trabalha com um modelo declarativo, onde descrevemos o estado desejado da nossa aplicação.
Para fazer isso, criamos um manifesto declarativo. 
Esse manifesto **não** é uma lista de comandos de como fazer com que a aplicação fique em um determinado estado, como é no 
modo imperativo. Mas sim, um documento onde falamos como deve ser o estado da aplicação.
Por exemplo, preciso que sempre tenham 3 réplicas da aplicação rodando.
O *Kubernetes* vai cuidar para que, caso uma réplica fique indisponível, outra fique em seu lugar e seja 
respeitada a quantidade de 3 réplicas sempre rodando.

![image](https://user-images.githubusercontent.com/9805258/124047493-b65c2980-d9ea-11eb-8a04-0f7ec1226c8b.png)


### Service

Nós aprendemos que *pods* podem ficar indisponíveis e quando isso acontece, 
*kubernetes* cuidará para que outro *pod* fique em seu lugar. Mas esse novo *pod* terá um novo IP,
então como ficam as aplicações que estavam acessando aquele endereço?

Podemos definir um *Service*, que será responsável por fornecer um IP e um nome DNS fixo para acessar um conjunto de *Pods*.
Assim, o *Kubernets* poderá balancear as *requests* entre os *Pods*.

<img src="https://user-images.githubusercontent.com/9805258/124047589-ec011280-d9ea-11eb-8110-ed68327b8aff.png" width="40%" height="40%">

### Labels
*Labels* são formas de identificar, organizar e selecionar objetos.
Por exemplo, eu posso ligar alguns *Pods* ao Serviço dando a eles o mesmo *Label*. Dessa forma, 
o serviço saberá quais *Pods* pertencem a ele. 

Observe que, no exemplo abaixo, todos os *Pods* com as *Labels* `prod` e `1.3` são reconhecidas pelo serviço.

![image](https://user-images.githubusercontent.com/9805258/124049130-33d56900-d9ee-11eb-961e-6ac18ad4b541.png)

Podemos usar os *Labels* também para fazer atualizações.

Por exemplo, temos os *Pods* com as *Labels* `prod` e `1.3` rodando e queremos incluir novos *Pods* com a versão `1.4`
mas sem parar os *Pods* da versão `1.3`. 

Como os *Pods* tem em comum a *Label* `prod`, desativamos a *Label* `1.3` do serviço e passamos
a reconhecer todos os *Pods* com a tag `prod`. Dessa forma, conseguimos rodar os *Pods* de ambas as versões agora.

![image](https://user-images.githubusercontent.com/9805258/124048999-ee18a080-d9ed-11eb-818d-fbba1112ae9d.png)


Porém, se eu quero remover os *Pods* da versão antiga do meu serviço, eu passo a reconhecer a *Label* `1.4`. Como os dois *Pods* da versão `1.3` não têm 
a *Label* da versão `1.4`, meu serviço passa a não reconhecer mais eles.

![image](https://user-images.githubusercontent.com/9805258/124049040-0ab4d880-d9ee-11eb-9982-fc879ead27fb.png)

Para começar o seu *Cluster*, dê uma olhadinha [aqui](https://kubernetes.io/docs/setup/production-environment/turnkey-solutions/), é possível encontrar uma lista com os principais providers e as instruções de uso de cada um.

### Referências

* [Getting Started with Kubernetes](https://app.pluralsight.com/library/courses/kubernetes-getting-started) by Nigel Poulton
* [Kubernetes Docs](https://kubernetes.io/docs/setup/)
