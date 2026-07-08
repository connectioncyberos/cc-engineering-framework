# DP-002 — Engineering Standards

## Objetivo

Aplicar o pacote DP-002 ao repositório `cc-engineering-framework`.

## Pré-requisito

Estar dentro do repositório local:

```powershell
cd C:\Projetos\connectioncyber\cc-engineering-framework
```

## Validação antes da cópia

```powershell
git status
```

O ideal é aparecer:

```text
nothing to commit, working tree clean
```

## Após copiar os arquivos

Execute:

```powershell
git status
git add .
git status
git commit -m "docs: add initial engineering standards"
git push
```

## Validação final

```powershell
git log --oneline -5
git status
```
