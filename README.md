# Beskar-cloud Flux CD infrastructures

Repository shows how to deploy complete OpenStack cloud on top of Kubernetes.

There are two separate cloud instances
 * [`shubham-cluster-1`](/clusters/shubham-cluster-1) ([base](/apps/base), [specific manifests](/apps/shubham-cluster-1))
   * OpenStack cloud with internal ceph (distributed storage part of infrastructure), authentication is done using LDAP and "Basic" HTTP authentication.
 * [`g2-oidc`](/clusters/g2-oidc) ([base+specific manifests](/apps/g2-oidc))
   * OpenStack cloud with connection to external ceph, authentication done using OIDC.

# Octavia Configuration

Explore the steps outlined below to configure Octavia, the OpenStack load balancing service. Follow these instructions to set up the Octavia management network, security groups, health manager, flavor for the amphora instance, key pair and Octavia Configuration Parameters.

## Octavia Management Network and Security Group

1. Create Octavia management network:
    ```bash
    openstack network create lb-mgmt-net -f value -c id
    ```

2. Create Octavia management subnet:
    ```bash
    openstack subnet create --subnet-range $OSH_LB_SUBNET --allocation-pool start=$OSH_LB_SUBNET_START,end=$OSH_LB_SUBNET_END --network lb-mgmt-net lb-mgmt-subnet -f value -c id
    ```

3. Create Octavia management security group:
    ```bash
    openstack security group create lb-mgmt-sec-grp
    ```

4. Configure security group rules for Octavia management:
    ```bash
    openstack security group rule create --protocol icmp lb-mgmt-sec-grp
    openstack security group rule create --protocol tcp --dst-port 22 lb-mgmt-sec-grp
    openstack security group rule create --protocol tcp --dst-port 9443 lb-mgmt-sec-grp
    ```

## Octavia Health Manager Security Group

1. Create security group for Octavia health manager:
    ```bash
    openstack security group create lb-health-mgr-sec-grp
    ```

2. Configure security group rule for Octavia health manager:
    ```bash
    openstack security group rule create --protocol udp --dst-port 5555 lb-health-mgr-sec-grp
    ```

## Ports for Octavia Health Manager

1. Create ports for health manager:
    ```bash
    CONTROLLER_IP_PORT_LIST=''
    CTRLS=$(kubectl get nodes -l openstack-compute-node=enabled -o name | awk -F"/" '{print $2}')
    for node in $CTRLS
    do
      PORTNAME=octavia-health-manager-port-$node
      openstack port create --security-group lb-health-mgr-sec-grp --device-owner Octavia:health-mgr --host=$node.cluster.local -c id -f value --network lb-mgmt-net $PORTNAME
      IP=$(openstack port show $PORTNAME -c fixed_ips -f value | awk -F',' '{print $1}' | awk -F'=' '{print $2}' | tr -d \')
      if [ -z $CONTROLLER_IP_PORT_LIST ]; then
        CONTROLLER_IP_PORT_LIST=$IP:5555
      else
        CONTROLLER_IP_PORT_LIST=$CONTROLLER_IP_PORT_LIST,$IP:5555
      fi
    done
    ```

## Amphora Instance Configuration

1. Create a flavor for amphora instance:
    ```bash
    openstack flavor create --id auto --ram 1024 --disk 2 --vcpus 1 --private m1.amphora
    ```

2. Create key pair to connect to the amphora instance via the management network:
    ```bash
    ssh-keygen -b 2048 -t rsa -N '' -f ~/.ssh/octavia_ssh_key
    openstack keypair create --public-key ~/.ssh/octavia_ssh_key.pub octavia_ssh_key
    ```

3. Download the Amphora image:
    ```bash
    curl https://tarballs.opendev.org/openstack/octavia/test-images/test-only-amphora-x64-haproxy-ubuntu-focal.qcow2 -o test-only-amphora-x64-haproxy-ubuntu-focal.qcow2
    ```

4. Create amphora image:
    ```bash
    openstack image create --disk-format qcow2 --container-format bare --public --file test-only-amphora-x64-haproxy-ubuntu-focal.qcow2 amphora
    ```

5. Set tag for amphora image:
    ```bash
    openstack image set --tag amphora <AMPHORA_IMAGE_ID>
    ```

## Octavia Configuration Parameters

### Controller Worker

Update the following parameters in your Octavia configuration:

```yaml
controller_worker:
  amp_image_owner_id: <IMAGE_ID>
  amp_secgroup_list: <SECURITY_GROUP_ID>
  amp_flavor_id: <FLAVOR_ID>
  amp_boot_network_list: <NETWORK_ID>
health_manager:
  bind_port: 5555
  bind_ip: 0.0.0.0
  controller_ip_port_list: "CONTROLLER_IP_PORT:5555,<CONTROLLER_IP_PORT>:5555"
```

Replace `<IMAGE_ID>`, `<SECURITY_GROUP_ID>`, `<FLAVOR_ID>`, and `<NETWORK_ID>` with the corresponding IDs:

`amp_image_owner_id:` Image ID of the created amphora image.  
`amp_secgroup_list:` Security group ID of the newly created security group for the amphora instance.  
`amp_flavor_id:` Flavor ID of the newly created amphora flavor.  
`amp_boot_network_list:` Network ID of the newly created amphora network.  
`controller_ip_port_list:` Port IPs of octavia-health-manager-ports.  
