helm create <chart_name>  --> creates a skeleton
helm install mychart ./chrt_name


Linting
helm lint .chart_name

helm temaplte mychart ./chartname --> renders the manifest locally
 use --debug to see the manifest


 helm install mychart ./chartname --dry-run -- k8s errors with installing

 helm package ./chart --> .tgz


 helm repo index ./tgz_folder --url <helm-repo>

 helm repo add mychart <helmrepo>