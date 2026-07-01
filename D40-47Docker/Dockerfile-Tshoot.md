## Dockerfile issue troubleshoot

#### to validate run below command
 ` docker build --check .`
- --check flag only validates Dockerfile structural syntax and formatting rules
####  Linter Way: Hadolint (Catches Shell Script Errors)
 `docker run --rm -i hadolint/hadolint < Dockerfile`

 #### for real time  build error run build and check the error 
 sudo docker build .
