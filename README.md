# Roteiro de estudo Kubernetes | KCNA e CKA

**Ponto de partida · ambiente validado, 0 de 40 exercícios concluídos.** Abra o Ubuntu e carregue o contexto conforme a seção “Começar uma sessão nesta máquina” do [guia do laboratório](LABORATORIO.md). Seu primeiro exercício é identificar os três nós e seguir o caminho de uma solicitação do kubectl.

## Como estudar

Use a mesma aplicação ao longo da trilha. Você vai publicar um catálogo HTTP, descobrir como o Kubernetes o mantém disponível e recuperar o serviço quando algo falhar. Para a KCNA, explique por que cada componente existe. Para a CKA, execute a tarefa em um terminal e demonstre o resultado.

1. Leia o enunciado e escreva em uma frase o comportamento que espera observar.
2. Escreva o YAML antes de consultar uma solução. Use `kubectl explain` para descobrir os campos e `kubectl apply --dry-run=server -f arquivo.yaml` para validar o manifesto contra a API.
3. Aplique no namespace do exercício. Compare estado desejado e observado com `get`, `describe`, eventos, logs ou uma requisição real.
4. Nas falhas propostas, registre o primeiro sintoma antes da correção. Altere uma variável, repita o teste e restaure a configuração de estudo.
5. Marque a caixa quando conseguir reproduzir a evidência descrita. Guarde os YAMLs e a conclusão em `modules/NN-tema/`. Se usar Git, faça um commit por desafio resolvido. Mantenha kubeconfig, tokens e dados reais fora do repositório.

**Ritmo sugerido:** 12 semanas, quatro sessões de 45–60 minutos por semana. Reserve a última sessão para repetir um exercício de memória. Avance por evidência se precisar de mais ou menos tempo.

| Semanas | Bloco | Entrega que marca a passagem |
|---|---|---|
| 1–2 | 01 · Fundamentos e agendamento | Explicar como um Deployment se torna Pods em execução e recuperar um Pod pendente |
| 3–4 | 02 · Rede e isolamento | Publicar o catálogo e provar quem consegue acessá-lo |
| 5 | 03 · Arquitetura e entrega cloud native | Resolver um simulado conceitual e justificar os erros |
| 6–7 | 04–06 · Configuração, identidade e persistência | Entregar configuração, permissão mínima e dados que sobrevivem à troca de Pod |
| 8–9 | 07–08 · Workloads, saúde e capacidade | Recuperar um rollout e explicar decisões de agendamento |
| 10–11 | 09 · Administração de cluster | Operar o ciclo de vida de um cluster e investigar falhas por camada |
| 12 | 10 · Simulado CKA e revisão | Entregar tarefas sob tempo e repetir as que ficaram incompletas |

## O que entra em cada prova

| KCNA · múltipla escolha | Peso | Onde treinar |
|---|---:|---|
| Fundamentos de Kubernetes | 44% | 01–08 e 13–16. Relacione um recurso ao componente que o gerencia. |
| Orquestração de contêineres | 28% | 03–12, 17–22 e 29–35. Distinga o papel do runtime, da rede, do armazenamento e do scheduler. |
| Entrega de aplicações cloud native | 16% | 25–28 e 32. Explique pipeline, reconciliação, telemetria e rollback. |
| Arquitetura cloud native | 12% | 25–28. Compare escalabilidade, resiliência e acoplamento em um cenário concreto. |

| CKA · tarefas no terminal | Peso | Onde treinar |
|---|---:|---|
| Arquitetura, instalação e configuração | 25% | 01–03, 30 e 36–38 |
| Workloads e agendamento | 15% | 04–12, 29 e 32–35 |
| Serviços e rede | 20% | 13–24 e 38 |
| Armazenamento | 10% | 31 e 39 |
| Diagnóstico e correção | 30% | 09, 12, 23–24, 33 e 37–40. Reserve tempo para explicar *qual evidência eliminou cada hipótese*. |

Pesos consultados em 18/11/2026. Antes de marcar as provas, confira a matriz publicada e a versão do ambiente CKA. [KCNA](https://www.cncf.io/training/certification/kcna/) · [CKA](https://www.cncf.io/training/certification/cka/) · [Currículos CNCF](https://github.com/cncf/curriculum)

## Módulo 01 · Fundamentos e scheduling

**Aquecimento, inclusive enquanto o ambiente é preparado.** Você pediu duas réplicas de um servidor HTTP. Um dos contêineres encerra, mas o Pod continua no nó. Mais tarde, alguém remove esse Pod inteiro. Quem age em cada caso? Anote sua previsão antes de abrir a resposta.

<details>
<summary>Conferir o raciocínio</summary>

No primeiro caso, o kubelet observa o encerramento e segue a política de reinício do Pod. Nos Pods de um Deployment, essa política é `Always`. Você pode observar o contador de reinícios aumentar com o mesmo UID de Pod. No segundo caso, o controlador do ReplicaSet percebe que falta uma réplica e solicita outro Pod à API. O scheduler escolhe um nó elegível, e o kubelet desse nó inicia os contêineres. O novo Pod tem outro UID. Nos exercícios 05 e 06, procure esses vínculos e identidades no seu cluster.

</details>

**Exercício 01, sua primeira sessão.** No Ubuntu, carregue o kubeconfig do laboratório conforme o guia. Execute os comandos abaixo. Em `modules/01-fundamentos/ex01.md`, anote o contexto, os nomes dos nós e onde estão o API server e o scheduler. Desenhe o caminho de uma solicitação do `kubectl` até o nó que executa o contêiner.

```bash
kubectl config current-context
kubectl get nodes -o wide
kubectl -n kube-system get pods -o wide
kubectl -n kube-system get daemonsets
```

- [ ] **01 · Reconhecer o cluster.** Identifique o control plane e os dois workers pelas saídas acima. Explique o papel de API server, etcd, scheduler, controller-manager e kubelet no seu desenho.
- [ ] **02 · Consultar a API.** Localize Pod, Deployment e Node com `api-resources` e `explain`. Registre grupo, versão e escopo de cada recurso.
- [ ] **03 · Isolar os estudos.** Escreva um Namespace chamado `catalogo` e um Pod simples nele. Localize o Pod consultando explicitamente seu namespace. Identifique imagem, runtime e nó.
- [ ] **04 · Declarar a aplicação.** Escreva um Deployment `catalogo` com duas réplicas de Nginx, porta 80 e label `app=catalogo`. Comprove que as duas ficaram disponíveis.
- [ ] **05 · Seguir os proprietários.** Partindo de um Pod, descubra seu ReplicaSet e o Deployment pelas `ownerReferences`. Explique quem cria um substituto quando uma réplica desaparece.
- [ ] **06 · Observar reconciliação.** Remova um Pod do seu Deployment. Registre os UIDs antes e depois e o número final de réplicas disponíveis.
- [ ] **07 · Escalar.** Altere o manifesto de duas para cinco réplicas e volte para duas. Compare réplicas desejadas, atuais e disponíveis durante a mudança.
- [ ] **08 · Selecionar objetos.** Crie uma label de ambiente no template de Pod. Liste os Pods por duas labels e confira o efeito do rollout.
- [ ] **09 · Investigar Pending.** Em um Pod separado, peça um `nodeSelector` sem correspondência. Localize o evento de scheduling e identifique a restrição insatisfeita.
- [ ] **10 · Direcionar um Pod.** Adicione a label correspondente a um worker de estudo e registre o nó escolhido para o Pod do exercício 09.
- [ ] **11 · Testar taint e toleration.** No worker rotulado, aplique um taint experimental `estudo=reservado:NoSchedule`. Compare dois Pods novos direcionados a ele, um com toleration e outro sem. Observe também o Pod que já estava lá.
- [ ] **12 · Reverter o experimento.** Remova o taint experimental, acompanhe o Pod pendente e retire a label experimental do nó. Explique por que tolerar um taint, por si só, não obriga o scheduler a escolher aquele nó.

## Módulo 02 · Services, DNS e NetworkPolicy

Use `catalogo` para a aplicação e `leitores` para clientes externos ao namespace. Descubra IPs, labels de DNS e EndpointSlices no seu cluster. Nos testes de política, abra uma conexão nova e use timeout curto. Guarde um teste positivo de referência antes de restringir tráfego.

- [ ] **13 · ClusterIP.** Escreva o Service `catalogo-http`, porta 80, para o Deployment. Confira seu selector e as portas publicadas.
- [ ] **14 · EndpointSlices.** Compare os endereços dos backends prontos com os IPs dos Pods. Escale a aplicação e observe a atualização.
- [ ] **15 · DNS e HTTP.** Crie um cliente BusyBox de longa duração em `catalogo`. Teste `nslookup catalogo-http` e `wget -T 3 -qO- http://catalogo-http`. Registre o que cada teste demonstra.
- [ ] **16 · Entre namespaces.** Repita de `leitores` usando `catalogo-http.catalogo.svc.cluster.local`. Explique por que o nome curto depende do namespace de origem.
- [ ] **17 · NodePort.** Crie um segundo Service na porta 30080. Do Pod cliente, consulte o IP interno de cada worker nessa porta. Registre os dois resultados.
- [ ] **18 · LoadBalancer.** Crie um terceiro Service e observe eventos e endereço externo. Identifique quem deveria alocar esse endereço. Em um kind sem provedor de balanceamento, ele pode ficar pendente.
- [ ] **19 · Provar enforcement.** Em um namespace temporário, confirme HTTP, aplique uma política que isole a entrada do servidor, teste novamente e remova a política. Só avance para 20–22 após observar sucesso → bloqueio → sucesso. Se o bloqueio faltar, confira seletores e suporte a NetworkPolicy do CNI instalado.
- [ ] **20 · Filtrar entrada.** Isole os Pods do catálogo e permita TCP 80 apenas para um cliente rotulado no mesmo namespace. Demonstre sucesso com o cliente autorizado e bloqueio com outro.
- [ ] **21 · Filtrar saída.** Isole o egress do cliente. Libere primeiro DNS em UDP/TCP 53 para os Pods DNS identificados em `kube-system`, depois HTTP para o catálogo. Em cada fase, teste resolução e HTTP por IP separadamente.
- [ ] **22 · Combinar seletores.** Permita a um leitor rotulado do namespace `leitores` acessar o catálogo. Combine os dois seletores no mesmo item de origem. Teste um Pod com a label em outro namespace e um Pod sem a label no namespace permitido.
- [ ] **23 · Diagnosticar falha.** Crie um Service de teste com selector divergente. Um cliente autorizado resolve o nome, mas HTTP falha. Inspecione DNS, EndpointSlices, readiness, portas e políticas. Corrija a causa e registre antes/depois.
- [ ] **24 · Desafio integrador.** Em namespaces novos, reconstrua catálogo, Service e clientes a partir dos seus manifestos. Entregue uma matriz de origem × destino × porta com resultado previsto e observado. Explique um bloqueio e uma permissão ponta a ponta.

## Módulo 03 · Arquitetura e entrega cloud native | fechamento KCNA

- [ ] **25 · Projetar.** Desenhe um catálogo com API sem estado e dados persistentes. O que acontece quando um Pod morre ou a demanda dobra? Compare replicação, autoscaling, alta disponibilidade e custo. Localize no desenho contêiner, registro de imagens, orquestrador e interfaces CRI, CNI e CSI.
- [ ] **26 · Entregar.** Desenhe o caminho commit → teste → imagem → manifesto → reconciliação. Compare CI, entrega contínua, implantação contínua e GitOps. Indique onde reverter uma versão em um fluxo GitOps.
- [ ] **27 · Observar.** Para uma requisição lenta, escolha uma métrica, uma linha de log e um trace. Diga que pergunta cada sinal responde e onde entram Prometheus e OpenTelemetry. Relacione um objetivo de disponibilidade a uma medida concreta.
- [ ] **28 · Simular a KCNA.** Faça 25 questões conceituais inéditas distribuídas pelos quatro domínios (11/7/4/3). Para cada erro, explique por que o distrator parecia correto. Meta de estudo sugerida de 20/25 em duas rodadas diferentes. Refaça o domínio com pior resultado.

## Módulos 04–08 · Administração de workloads

- [ ] **29 · M04, ConfigMaps e Secrets.** Entregue configuração ao catálogo por variável e por volume. Use um valor fictício para o Secret. Altere a configuração e compare quando a aplicação enxerga a atualização em cada caso.
- [ ] **30 · M05, RBAC.** Crie uma ServiceAccount leitora, Role e RoleBinding no namespace. Com `kubectl auth can-i --as=system:serviceaccount:catalogo:leitora`, prove que pode listar Pods ali e que não pode apagá-los nem listá-los em outro namespace. Explique quando usar ClusterRole.
- [ ] **31 · M06, armazenamento.** Inspecione StorageClass, provisionador, PV e PVC. Grave uma frase em um volume, substitua o Pod e leia a frase. Relacione modo de acesso, política de recuperação e afinidade de nó à persistência observada.
- [ ] **32 · M07, controladores e rollout.** Compare Deployment, StatefulSet, DaemonSet, Job e CronJob escolhendo um caso para cada. Atualize a imagem do catálogo, acompanhe o rollout e demonstre o rollback pela imagem efetivamente em execução.
- [ ] **33 · M07, saúde.** Em uma cópia do catálogo, configure uma readiness probe com caminho incorreto. Observe Ready, reinícios e condição dos endpoints. Corrija e repita. Explique como liveness e startup probes mudariam a investigação.
- [ ] **34 · M08, capacidade.** Use requests de CPU que nenhum worker possa atender e recupere o Pod. Compare requests e limits. Esboce um HPA e identifique o pré-requisito de métricas para observar escalabilidade real.
- [ ] **35 · M08, distribuição.** Espalhe réplicas entre os workers com anti-affinity ou topology spread. Compare comportamento obrigatório e preferencial quando faltar capacidade. Demonstre a distribuição consultando os nós.

## Módulo 09 · Operar e recuperar clusters

- [ ] **36 · Instalação e extensões.** Em VMs Linux descartáveis ou em um simulador CKA com acesso aos nós, prepare runtime, kubelet e kubeadm. Inicialize o control plane, instale um CNI compatível e associe um worker. Verifique os nós e desenhe a extensão para alta disponibilidade. O kind já entrega esses passos automatizados.
- [ ] **37 · Manutenção.** Nesse segundo ambiente, planeje e execute um upgrade suportado de uma versão minor por vez, com drain e uncordon. Registre versões e disponibilidade antes/depois. Faça snapshot de etcd e ensaie restauração em uma cópia descartável. Verifique a presença de um objeto conhecido após a recuperação.
- [ ] **38 · APIs e extensibilidade.** No kind, renderize uma aplicação com Kustomize, instale um chart com Helm e inspecione uma CRD com seu recurso. Explique o papel de um operador. Em ambiente com controlador compatível, publique HTTP com Gateway e HTTPRoute, verifique condições e tráfego. Compare com Ingress e IngressClass.
- [ ] **39 · Plantão de diagnóstico.** Resolva três chamados em recursos de treino: PVC pendente, contêiner em reinício e nó indisponível. Para o último, use o ambiente descartável do exercício 36. Correlacione eventos, logs atuais/anteriores, kubelet e runtime. Entregue sintoma, evidência, correção mínima e reteste para cada chamado.

## Módulo 10 · Revisão CKA

- [ ] **40 · Simulado prático.** Reserve duas horas. Peça uma rodada inédita com tarefas nos cinco domínios. Inclua recuperar uma aplicação, rotear HTTP, obter persistência, corrigir RBAC e devolver um nó ao serviço. Registre o tempo até cada validação e o estado final. Refaça as tarefas incompletas em uma nova sessão sem consultar a primeira solução.

## Registro para retomar na próxima sessão

| Data | Exercício | Hipótese ou previsão | Evidência observada | Próxima ação |
|---|---|---|---|---|
| — | 01 · a iniciar | — | — | Validar os três nós e identificar os componentes |

Para estudar com acompanhamento, use: “Estou no exercício 01 do meu roteiro. Apresente o desafio, aguarde minha tentativa e revise meu raciocínio antes de mostrar uma solução.” Nas sessões seguintes, inclua a última linha do registro e a saída relevante.

