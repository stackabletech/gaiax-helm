# Setup of kind with ingress

```bash
# Create kind cluster with extra config
cat <<EOF | kind create cluster --config=-
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
  kubeadmConfigPatches:
  - |
    kind: InitConfiguration
    nodeRegistration:
      kubeletExtraArgs:
        node-labels: "ingress-ready=true"
  extraPortMappings:
  - containerPort: 80
    hostPort: 80
    protocol: TCP
  - containerPort: 443
    hostPort: 443
    protocol: TCP
EOF

# Deploy nginx ingress
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```

More details: https://kind.sigs.k8s.io/docs/user/ingress/

# Configuration of the IDP Bridge

Create an OIDC client for keycloak specifying any additional scopes to use for this OIDC client.

POST $IDP_BRIDGE/oidc/clients

```json
{
  "client_name": "kc",
  "redirect_uris": ["https://$KEYCLOAK/realms/test/broker/idp-bridge/endpoint"],
    "scopes": ["openid", "profile"]
}
```

POST $IDP_BRIDGE/oidc/clients

```json
{
  "client_name": "kc",
  "redirect_uris": ["https://$KEYCLOAK/realms/test/broker/idp-bridge/endpoint"],
    "scopes": ["openid", "profile", "gaiax"]
}
```

Create a new scope that should be mapped by the idp-bridge.

POST $IDP_BRIDGE/oidc/scopes

```json
{
  "name": "profile"
}
```

POST $IDP_BRIDGE/oidc/scopes

```json
{
  "name": "profile"
}
```

Create a list of mappings from the presented VC to OIDC claims in this scope

POST $IDP_BRIDGE/oidc/scopes/mappings?scopeName=profile

```json
[
  {
    "token_claim": "username",
    "credential_type": "VerifiableId",
    "credential_format": "jwt_vc",
    "claim_path": "$.vc.credentialSubject.id"
  },
  {
    "token_claim": "family_name",
    "credential_type": "VerifiableId",
    "credential_format": "jwt_vc",
    "claim_path": "$.vc.credentialSubject.familyName"
  },
  {
    "token_claim": "given_name",
    "credential_type": "VerifiableId",
    "credential_format": "jwt_vc",
    "claim_path": "$.vc.credentialSubject.firstName"
  },
  {
    "token_claim": "gender",
    "credential_type": "VerifiableId",
    "credential_format": "jwt_vc",
    "claim_path": "$.vc.credentialSubject.gender"
  },
  {
    "token_claim": "birthdate",
    "credential_type": "VerifiableId",
    "credential_format": "jwt_vc",
    "claim_path": "$.vc.credentialSubject.dateOfBirth"
  },
  {
    "token_claim": "address",
    "credential_type": "VerifiableId",
    "credential_format": "jwt_vc",
    "claim_path": "$.vc.credentialSubject.currentAddress[0]"
  }
]
```
