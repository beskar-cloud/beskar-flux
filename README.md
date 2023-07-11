# Beskar-cloud Flux CD infrastructures

Repository shows how to deploy complete OpenStack cloud on top of Kubernetes.

There are two separate cloud instances
 * [`shubham-cluster-1`](/clusters/shubham-cluster-1) ([base](/apps/base), [specific manifests](/apps/shubham-cluster-1))
   * OpenStack cloud with internal ceph (distributed storage part of infrastructure), authentication is done using LDAP and "Basic" HTTP authentication.
 * [`g2-oidc`](/clusters/g2-oidc) ([base+specific manifests](/apps/g2-oidc))
   * OpenStack cloud with connection to external ceph, authentication done using OIDC.

