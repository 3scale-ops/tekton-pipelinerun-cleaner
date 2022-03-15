# Tekton Pipelun Cleaner

A simply bash script running in an Alpine docker, to clean up completed pipeline runs, keeping up to `NUM_TO_KEEP` instances for each Pipeline.

```bash
...
- name: kubectl
  image: docker.io/alpine/k8s:1.20.7
  env:
    - name: NUM_TO_KEEP
      value: "3"
  command:
    - /bin/bash
    - -c
    - >
      while read -r PIPELINE; do
        while read -r PIPELINE_TO_REMOVE; do
          test -n "${PIPELINE_TO_REMOVE}" || continue;
          kubectl delete ${PIPELINE_TO_REMOVE} \
              && echo "$(date -Is) PipelineRun ${PIPELINE_TO_REMOVE} deleted." \
              || echo "$(date -Is) Unable to delete PipelineRun ${PIPELINE_TO_REMOVE}.";
        done < <(kubectl get pipelinerun -l tekton.dev/pipeline=${PIPELINE} --sort-by=.metadata.creationTimestamp -o name | head -n -${NUM_TO_KEEP});
      done < <(kubectl get pipelinerun -o go-template='{{range .items}}{{index .metadata.labels "tekton.dev/pipeline"}}{{"\n"}}{{end}}' | uniq);
```

## Deplyment

```bash
# rbac
kubectl -f rbac.yaml
# cronjob
kubectl -f cronjob.yaml
```

## Notes

- Related to https://github.com/tektoncd/experimental/issues/479