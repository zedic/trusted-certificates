# How to Import trusted ssl certificates
# This should resolve issues when using Self-Signed or Custom certificates on self managed installs of GitLab CE or EE 

### [https://github.com/zedic/trusted-certificates](https://github.com/zedic/trusted-certificates)

## Add a Self-Signed Certificate to Trusted Certificate Store

### Instructions for on Windows & Ubuntu / Debian Linux

### Configure Git on Windows to use the Windows Trusted Certificate Store

## Windows

#### Export the certificate
 - The following commands will export a certificate from a remote https website to a file.
 - Update TARGET and OUTPUT_PATH if needed.
 - Run the commands in **Powershell 5**
 - NOTE: For some reason it will fail in Powershell 7
~~~
# Update TARGET and OUTPUT_PATH as desired. 
$TARGET="https://example.com"
$OUTPUT_PATH="$HOME\example.com.crt"

$webRequest = [Net.WebRequest]::Create("$TARGET")
try { $webRequest.GetResponse() } catch {}
$cert = $webRequest.ServicePoint.Certificate
$bytes = $cert.Export([Security.Cryptography.X509Certificates.X509ContentType]::Cert)
set-content -value $bytes -encoding byte -path $OUTPUT_PATH
~~~

#### Use certmgr.msc to import the certificate
  - Press Win + R on the keyboard
  - Run certmgr.msc
  - Select Trusted Root Certification Authorities
  - Right click Certificates
  - Select Import
  - Browse to the certificate file you exported in the step above.

![certmgr.png](https://github.com/zedic/trusted-certificates/blob/main/certmgr.png?raw=true)

#### By default, git uses the OpenSSL Certificate store.  This command configures git to use the local windows certificate store for SSL verification.
  
  - Open a Powershell and run the following commands
~~~ 
git config --global http.sslBackend schannel
~~~

## Ubuntu / Debian / WSL (Windows Subsystem for Linux)

~~~ 
# https://ubuntu.com/server/docs/install-a-root-ca-certificate-in-the-trust-store

# Gets the Self Signed certificate from a remote host
# Copies cert to /usr/local/share/ca-certificates
# Updates the Trusted Certificates using the command update-ca-certificates

# Update this Variables as needed
export TARGET_HOST=https://example.com
export OUTPUT_FILE=example.com.crt
export OUTPUT_PATH=$HOME/$OUTPUT_FILE

# Installing ca-certificates
echo ""
echo "*** Installing ca-certificates package ***"
sudo apt install -y ca-certificates

# Getting Cert from remote host
echo ""
echo "*** Getting Certificate from $TARGET_HOST ***"
openssl s_client -showcerts -connect $TARGET_HOST:443 -servername $TARGET_HOST  </dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > $OUTPUT_PATH

# Copying cert to /usr/local/share/ca-certificates
echo ""
echo "*** Copying cert to /usr/local/share/ca-certificates ***"
sudo cp $OUTPUT_PATH /usr/local/share/ca-certificates

# Running update-ca-certificates 
echo ""
echo "*** Running update-ca-certificates ***"
sudo update-ca-certificates -v

echo ""
echo "*** Testing Cert ***"
curl https://$TARGET_HOST
echo ""
~~~
