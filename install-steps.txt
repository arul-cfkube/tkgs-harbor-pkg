kubectl create clusterrolebinding default-tkg-admin-privileged-binding --clusterrole=psp:vmware-system-privileged --group=system:authenticated
tanzu package available list -A
#tanzu package install cert-manager --package-name cert-manager.tanzu.vmware.com --namespace cert-manager --version 1.5.3+vmware.2-tkg.1 --create-namespace
tanzu package install contour --package-name=contour.tanzu.vmware.com --version=1.18.2+vmware.1-tkg.1 --values-file=contour-lb.yml --create-namespace


image_url=$(kubectl -n tanzu-package-repo-global get packages harbor.tanzu.vmware.com.2.3.3+vmware.1-tkg.1 -o jsonpath='{.spec.template.spec.fetch[0].imgpkgBundle.image}')
imgpkg pull -b $image_url -o /tmp/harbor-package-2.3.3+vmware.1-tkg.1
cp /tmp/harbor-package-2.3.3+vmware.1-tkg.1/config/values.yaml harbor-data-values.yaml

#brew install yq and run the cmd to remove all comments
yq -i eval '... comments=""' harbor-data-values.yaml

## update the data values... secrets must be 16 len and xrsf key should be 32 len

#install
tanzu package install harbor \
--package-name harbor.tanzu.vmware.com \
--version 2.3.3+vmware.1-tkg.1 \
--values-file harbor-data-values.yaml \
--create-namespace

#update
tanzu package installed update harbor \
--package-name harbor.tanzu.vmware.com \
--version 2.3.3+vmware.1-tkg.1 \
--values-file  harbor-data-values.yaml \
--create-namespace


tanzu package installed list -A
- Retrieving installed packages... I0718 15:36:22.405042   50551 request.go:665] Waited for 1.045173875s due to client-side throttling, not priority and fairness, request: GET:https://10.220.18.166:6443/apis/cert-manager.io/v1?timeout=32s


  NAME          PACKAGE-NAME                   PACKAGE-VERSION        STATUS               NAMESPACE
  cert-manager  cert-manager.tanzu.vmware.com  1.5.3+vmware.2-tkg.1   Reconcile succeeded  cert-manager-90d7a4af
  contour       contour.tanzu.vmware.com       1.18.2+vmware.1-tkg.1  Reconcile succeeded  contour-7ff91622
  harbor        harbor.tanzu.vmware.com        2.3.3+vmware.1-tkg.1   Reconciling          harbor-14596ab8


kubectl create secret generic harbor-database-redis-trivy-jobservice-registry-image-overlay -o yaml --dry-run=client --from-file=harbor-vsphsere-overlay.yml | kubectl apply -f -

kubectl annotate packageinstalls harbor  ext.packaging.carvel.dev/ytt-paths-from-secret-name.1=harbor-database-redis-trivy-jobservice-registry-image-overlay



kubectl get svc envoy -n tanzu-system-ingress -o jsonpath='{.status.loadBalancer.ingress[0]}'
