# Lab 5
## 1. Build the vulnerable image

```bash
git clone https://github.com/Ibrahim-Adel15/TrivyImage.git
cd TrivyImage
docker build -t trivy-lab:v1 .
```
## 2. Install Trivy and verify

```bash
curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sudo sh -s -- -b /usr/local/bin
trivy --version
```

## 3. Scan

```bash
trivy image trivy-lab:v1                                    # all severities
trivy image --severity HIGH,CRITICAL trivy-lab:v1           # only HIGH and CRITICAL
trivy image --severity HIGH,CRITICAL --format template \
  --template "@/usr/local/share/trivy/templates/html.tpl" \
  -o report-v1.html trivy-lab:v1                            # HTML report
```

## 4. Results for trivy-lab:v1

| Item | Value |
|------|-------|
| Base OS | Debian 11.11 (bullseye) |
| HIGH | 2148 |
| CRITICAL | 222 |
| With a fix available | 1114 HIGH, 183 CRITICAL |

## 5. Remediation (ivolve-app:v2)

Changes (see `Dockerfile` and `package.json`):

```bash
docker build -t ivolve-app:v2 .
docker run -d --name test -p 3000:3000 ivolve-app:v2
curl localhost:3000 && curl localhost:3000/health
docker rm -f test
```

## 6. Rescan and verify

```bash
trivy image --severity HIGH,CRITICAL --ignore-unfixed --format template \
  --template "@/usr/local/share/trivy/templates/html.tpl" \
  -o report-v2.html ivolve-app:v2
trivy image --severity HIGH,CRITICAL --ignore-unfixed --exit-code 1 ivolve-app:v2
echo $?    # 0 = no fixable HIGH/CRITICAL vulnerabilities remain
```

| Image | HIGH | CRITICAL |
|-------|------|----------|
| trivy-lab:v1 | 2148 | 222 |
| ivolve-app:v2 | _fill in from report-v2.html_ | _fill in_ |
