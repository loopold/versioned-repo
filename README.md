# Versioned Repo
Demo purposes 

## Settings

We use options described in [github-tag-action] repo: 
- `DEFAULT_BUMP` (optional) - `patch` - Which type of bump to use when none explicitly provided (default: `minor`).


## Workflow logic

```mermaid
flowchart TD
  j_env[[**RESOLVE-ENV**]] --> ENV{ENV}
  ENV --> |dev| DEV["`DEV_SCRIPTS_PATH
    DEV_SERVER_IP`"]
  ENV --> |prod| PROD["`PROD_SCRIPTS_PATH
    PROD_SERVER_IP`"]

  j_tag[[**TAG-VERSION**]] --> |pull_request| REF{base_ref}
  REF --> |develop| PRE[1.0.0-pre.1]
  REF --> |main| SEM[1.0.0]
```
```mermaid
flowchart TD
  j_build[[BUILD-AND-PUSH]] 
  --> SHORT(Create SHA short)
  --> ECR(Login to AWS ECR)
  --> TAG{tag-version?}
  TAG --> |skipped| IMGSHA[:6514fcb] --> PUSH
  TAG --> |success| IMGVER["`:1.0.0
    :1.0.0-pre.1
    :latest`"] --> PUSH
  

  j_deploy[[**DEPLOY**]] --> |always| B{IMAGE_TAG}
  B --> |input| C
  B --> |build-and-push| C
  B --> |latest| C
  C(Deploy to AWS)
  script["`prune
    replace image in yaml
    pull & up -d`"]
  C --> |cd $ENV.scripts_path| script
```


## Troubleshooting

```
pre_release = false
fatal: ambiguous argument 'develop..HEAD': unknown revision or path not in the working tree.
Use '--' to separate paths from revisions, like this:
'git <command> [<revision>...] -- [<file>...]'
```
line 163:
```sh
declare -A history_type=(
    ["last"]="$(git show -s --format=%B)" \
    ["full"]="$(git log "${default_branch}"..HEAD --format=%B)" \
    ["compare"]="$(git log "${tag_commit}".."${commit}" --format=%B)" \
)
```

---
[github-tag-action]: https://github.com/anothrNick/github-tag-action/blob/master/README.md#options
