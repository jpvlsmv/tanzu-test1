# tanzu-test1

Consider that I need to run more than one instance of a Helm-packaged application, each with slightly different configurations.

These could be a DEV and PROD instance with different number of replicas set in values.yaml, or an internal- and external-facing webserver that have slightly different policies applied to the same application stack.

I want to manage these with as little administrative overhead as possible, with as much automation as is available in the Infrastructure-Service platform, and to follow a git-like workflow.

So far, I have abstracted three sets of values that I need to be able to scalably manage across the global environment:
1) The "base" settings defining how the application needs to be reconfigured to operate correctly
2) The "company" settings that tell the application how to interact with other infrastructure that we manage
3) The "instance" settings that specify the unique attributes of a specific unit

The inheritance of these settings should be transitive, with more-specific overrides and intelligent merging, and should not be overly verbose/explicit.

## Multiple Helm releases
This can be accomplished natively within Helm, by identifying Chart dependencies and passing configuration values around:

* base-chart depends on the vendor-provided chart and specifies the base Values.yaml settings (which gets packaged into a repository including charts/vendorchart.tgz)
* company-chart depends on base-chart and overrides company settings to that Values.yaml (which gets packaged into a repository including charts/base.tgz, which includes charts/base/charts/vendor.tgz)
* I `helm install company-chart --values=instance.yaml` into my cluster (Or I can create an instance-chart that recurses the above step)

This is fairly easy to implement in a git release workflow.

## Kustomize
Kustomize can put these sorts of overlays onto Kubernetes resources, but Helm charts aren't treated as first-class objects, so some workarounds are required, as implemented in ./{base,disw,tanzu-poc-wv}/kustomization.yaml.  It requires adding `--enable-helm` and disabling the file loader restrictions, but it results in an instance with the correct configuration.

## Continuous Delivery
This should be a very straightforward application of CD principles, and I would expect any enterprise-grade Infrastructure-Service platform to have a ready description of how their tool/interface would implement it, at least as a familiar "good-practice" wrapper around Flux or ArgoCD.
