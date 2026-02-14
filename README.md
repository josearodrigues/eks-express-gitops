# GitOps

Estado desejado do Kubernetes para o EKS Express.

## Índice

- [Escopo](#escopo)
- [Estrutura do Diretório](#estrutura-do-diretório)
- [Pré-requisitos](#pré-requisitos)
- [Modelo de Reconciliação](#modelo-de-reconciliação)
- [Regras de Autoria](#regras-de-autoria)
- [Promoção e Rollback](#promoção-e-rollback)
- [Validação](#validação)
- [Checklist de Pull Request](#checklist-de-pull-request)
- [Observabilidade Operacional](#observabilidade-operacional)
- [Contribuição](#contribuição)
- [Troubleshooting](#troubleshooting)

## Escopo

Este diretório é a fonte de verdade dos manifests aplicados pelo Argo CD.

## Estrutura do Diretório

```text
gitops/
  argocd/application.yml
  backend/
    deploy.yml
    service.yml
  frontend/
    deploy.yml
    service.yml
  kustomization.yml
```

## Pré-requisitos

- Cluster Kubernetes com Argo CD instalado.
- Acesso do Argo CD ao repositório (SSH key ou credenciais HTTPS).
- `kustomize` para renderização local dos manifests.
- Opcional: `kubeconform` para validação de schema.

## Modelo de Reconciliação

1. CI publica nova imagem de container.
2. CI atualiza a tag em `kustomization.yml`.
3. Argo CD detecta drift entre cluster e Git.
4. Argo CD sincroniza os recursos para refletir o repositório.

## Regras de Autoria

- Toda alteração de workload em produção deve passar por Git.
- Não editar manualmente no cluster recursos gerenciados por GitOps.
- Usar tags imutáveis de imagem (preferencialmente SHA de commit).
- Manter manifests idempotentes e explícitos.

## Promoção e Rollback

Promoção:

1. Merge de mudança aprovada para `main`.
2. CI atualiza referências de imagem.
3. Argo CD sincroniza namespace de destino.
4. Validar saúde do deployment e acessibilidade do serviço.

Rollback:

1. Reverter o commit que introduziu o estado indesejado.
2. Confirmar Argo CD como `Healthy` e `Synced`.
3. Validar endpoints da aplicação após rollback.

## Validação

```bash
kustomize build .
```

Validação de schema (opcional):

```bash
kustomize build . | kubeconform -strict -summary
```

## Checklist de Pull Request

- Seletores de Deployment e Service estão consistentes.
- Porta do container, Service e rota de ingresso estão alinhadas.
- `resources.requests` e `resources.limits` estão definidos.
- `readinessProbe` e `livenessProbe` estão definidos.
- Não existem credenciais ou tokens fixos no código.

## Observabilidade Operacional

- Monitorar eventos de sync e diff no Argo CD.
- Acompanhar rollout e reinícios de pods (`kubectl rollout status`, `kubectl get pods`).
- Confirmar digest da imagem efetivamente em execução a cada release.

## Contribuição

Fluxo recomendado para contribuições:

1. Criar branch a partir de `main`.
2. Alterar manifests de forma incremental e idempotente.
3. Validar com `kustomize build .` antes do commit.
4. Abrir pull request com diff esperado de deploy/rollback.
5. Após merge, acompanhar sincronização e saúde no Argo CD.

## Troubleshooting

- Problema de sync: validar `argocd/application.yml` (repo URL, path e destination).
- Rollback não efetivo: confirmar reversão do commit correto e novo sync.
- Imagem divergente: validar atualização de tag em `kustomization.yml` e status no Argo CD.
