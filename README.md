# crossplane-eksauto

Steps in this repo deploy Crossplane v2 in a Kind Cluster and provision EKS Auto Mode as a target cluster on an AWS account. 

The only prerequisites are an Ubuntu-based VM and admin access to an AWS account. 

### Clone the Repo
```
git clone https://github.com/moonorb/crossplane-eksauto.git
cd crossplane-eksauto
```

### Install Podman
```
sudo apt update
sudo apt install -y podman
podman --version
```
### Install and Run Nix Shell
Nix shell is used to install kind, kubectl, and awscli.
```
sh <(curl -L https://nixos.org/nix/install) --no-daemon
. ~/.nix-profile/etc/profile.d/nix.sh
echo ". ~/.nix-profile/etc/profile.d/nix.sh" >> ~/.bashrc
nix-shell scripts/shell.nix --run $SHELL
```

To get back to nix shell next time, just run the last line above. 

### Update Your AWS Credentials File
```
cat <<EOF > credentials/aws-credentials.txt
[default]
aws_access_key_id = AKIAZQ3DPHSQIF4FB22C
aws_secret_access_key = iB5/ppehdnkLT4RMfh+NCT6zFpG8b8UGIXtRJ5xM
# These creds are fake...
EOF
```

## Run Installation Script
This script will install a single-node Kubernetes management cluster via kind as well as Crossplane controllers and required providers on this cluster. It takes approximately 4-5 minutes for all the pods to be ready. 
```
./scripts/install-crossplane.nix
```

## Apply Crossplane XRD, Composition and Composite Resource
```
kubectl apply -f api-xr/definition.yaml 
compositeresourcedefinition.apiextensions.crossplane.io/xeks.aws.moonorb.com created

kubectl apply -f api-xr/composition.yaml 
composition.apiextensions.crossplane.io/xeksauto.aws.moonorb.com created

kubectl apply -f api-xr/xr.yaml 
xeks.aws.moonorb.com/sandbox-eksauto created
```

## Check the Status of the Cluster (Composite Resource)
```
watch "crossplane beta trace XEKS sandbox-eksauto"

NAME                                                                       SYNCED   READY   STATUS
XEKS/sandbox-eksauto                                                       True     True    Available
├─ SecurityGroupIngressRule/sandbox-eksauto-sg-ingress-0                   True     True    Available
├─ SecurityGroup/sandbox-eksauto-e768d7a25fae                              True     True    Available
├─ AccessEntry/sandbox-eksauto-5ea3bca89387                                True     True    Available
├─ AccessPolicyAssociation/sandbox-eksauto-3585ae94083b                    True     True    Available
├─ ClusterAuth/sandbox-eksauto-71980c7b363d                                True     True    Available
├─ Cluster/sandbox-eksauto                                                 True     True    Available
├─ ProviderConfig/sandbox-eksauto-helm                                     -        -
├─ InstanceProfile/amazon-eks-auto-node-instance-profile-sandbox-eksauto   True     True    Available
├─ OpenIDConnectProvider/sandbox-eksauto-e655d24b6d18                      True     True    Available
├─ RolePolicyAttachment/sandbox-eksauto-56a861228c2a                       True     True    Available
├─ RolePolicyAttachment/sandbox-eksauto-705355a50511                       True     True    Available
├─ RolePolicyAttachment/sandbox-eksauto-b55c10381d72                       True     True    Available
├─ RolePolicyAttachment/sandbox-eksauto-b92caebfbfa6                       True     True    Available
├─ RolePolicyAttachment/sandbox-eksauto-cdd44c8e3403                       True     True    Available
├─ RolePolicyAttachment/sandbox-eksauto-d692c80079c6                       True     True    Available
├─ RolePolicyAttachment/sandbox-eksauto-e1d46f6388dd                       True     True    Available
├─ Role/amazon-eks-auto-cluster-role-sandbox-eksauto                       True     True    Available
├─ Role/amazon-eks-auto-node-role-sandbox-eksauto                          True     True    Available
├─ ProviderConfig/sandbox-eksauto                                          -        -
├─ Object/sandbox-eksauto-aws-auth                                         True     True    Available
└─ Object/sandbox-eksauto-irsa-settings                                    True     True    Available
