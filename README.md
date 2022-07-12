# bootstrap-vbox

Requires Virtualbox, which can be downloaded from here: https://www.virtualbox.org/

## Tools

- Vault CLI: https://www.vaultproject.io/downloads.html
- Minio CLI: brew install minio/stable/mc
- BOSH CLI:  brew install cloudfoundry/tap/bosh-cli
- jq:        brew install jq

## Initial setup

```bash
mkdir workspace
cd ~/workspace
git clone https://github.com/jmcclenny-pre/bootstrap-vbox.git
sudo route add -net 10.244.0.0/16 192.168.50.6 # Mac OS X
```
# Without TLS (using system generated self-signed certs or plaintext http communication)
- First run `./bootstrap_vbox.sh --initial`
- Subsequent `./bootstrap_vbox.sh`

## Untrusted URL's

```bash
Vault     https://10.244.16.2
Concourse http://10.244.16.3:8080
Minio     http://10.244.16.4:9000
```

# With TLS (using CA signed PKI certs for communication with Transport Layer Security (TLS))
Provide requisite certificate values and configuration in the following files:
- bootstrap-vbox/bosh/params/bosh-params.yml
   * trusted_certs: All of the CA certificates used for the PKI certs being applied to the managed systems
- bootstrap-vbox/concourse/params/concourse-params.yml
   * external_url: https://\<CONCOURSE_FQDN\>
   * concourse_fqdn: CN from the public certificate
   * ca: CA that signed certificate for concourse web server
   * certificate: Signed public certificate for concourse web server
   * private_key: Private key for concourse web server
- bootstrap-vbox/minio/params/minio-params.yml
   * minio_ca_cert: CA that signed certificate for minio web server
   * minio_certificate: Signed public certificate for minio web server
   * minio_private_key: Private key for minio web server
- bootstrap-vbox/vault/params/vault-params.yml
   * vault_ca_cert: CA that signed certificate for vault web server
   * vault_pki_cert: Signed public certificate for vault web server
   * vault_pki_key: Private key for vault web server

Execute with the `--tls` flag
- First run `./bootstrap_vbox.sh --initial --tls`
- Subsequent `./bootstrap_vbox.sh -tls`

## Trusted URL's

```bash
Vault     https://10.244.16.2 or https://<VAULT_FQDN>
Concourse https://10.244.16.3 or https://<CONCOURSE_FQDN>
Minio     https://10.244.16.4 or https://<MINIO_FQDN>
```



## Alias your environment

```bash
bosh alias-env vbox -e 192.168.50.6 --ca-cert <(bosh int ~/deployments/vbox/bosh-creds.yml --path /director_ssl/ca)
```

## Writing credentials to vault

```bash
export VAULT_ADDR=https://10.244.16.2
export VAULT_SKIP_VERIFY=true
root_token=$(cat ~/deployments/vbox/tokens.json | jq -r '.root_token')

vault write concourse/common/<key_name> value="<value>"
vault write concourse/<team_name>/<key_name> value"<value>"
vault write concourse/<team_name>/<pipeline_name>/<key_name> value"<value>"
```

## Known Issues

- Time for some reason doesn't sync on minio server, need to run the `fix_minio_time.sh` script in order to use the Minio Client CLI.

- The route doesn't survive a reboot. Researching if it is even worth persisting.

- Sometimes concourse deployment fails, just re-run the script without `--initial` flag.
