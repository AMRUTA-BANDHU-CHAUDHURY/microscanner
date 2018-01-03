# MicroScanner
WORK IN PROGRESS - Scan your container images for vulnerabilities

## Overview
Aqua Security's *microscanner* lets you check your container images for vulnerabilities. If your image (including any of its dependencies) has any known high-severity issue, microscanner can fail the image build, making it easy to include as a step in your CI/CD pipeline. 

## Registering for a token
To use microscanner you'll first need to register for a token. 

```
$ docker run --rm -it aquasec/microscanner --register <email address>
```

We'll send a microscanner token to the email address you specify. 

This process will prompt you to agree to the [Terms and Conditions for microscanner]() **TODO!!**.

## Running microscanner 
The microscanner is designed to be run as part of building a container image. You add the microscanner executable into the image, and a step to run the scan, which will examine the contents of the image filesystem for vulnerabilities. If high severity vulnerabilities are found, this will fail the image build (though you can force the scanner to exit with zero by setting the ```--continue-on-failure``` flag).

### Adding microscanner to your Dockerfile
The following lines add microscanner to a Dockerfile, and execute it.
```
ADD https://get.aquasec.com/microscanner
RUN chmod +x microscanner
RUN microscanner <TOKEN> [--continue-on-failure]
```

### Windows version
There is also a Windows version of the executable, which is added to a Dockerfile in a similar way. **TODO!!** Check this
```
ADD https://get.aquasec.com/microscanner.exe
RUN microscanner.exe <TOKEN> [--continue-on-failure]
```

### Example 
Example Dockerfile
```
FROM debian:jessie-slim
RUN apt-get update && apt-get -y install ca-certificates
ADD https://get.aquasec.com/microscanner
RUN chmod +x microscanner
ARG token
RUN /microscanner ${token}
RUN echo "No vulnerabilities!"
```
Pass the token obtained on registration in at build time.
```
$ docker build --build-arg=token=<TOKEN> .
```
### Continue on failure
Specifying the ```--continue-on-failure``` flag allows you to continue the build even if high severity issues are found. **TODO!!** Are the results logged out as part of the build? 

## Best practices 

* Since the token is a [secret value](https://blog.aquasec.com/managing-secrets-in-docker-containers), it's a good idea to pass this in as a build argument rather than hard-coding it into your Dockerfile. 
* The step that runs microscanner needs to appear in your Dockerfile after you have added or built files and directories for the container image. Build steps happen in the order they are defined in the Dockerfile, so anything that gets added to the image after the microscanner is run won't be scanned for vulnerabilities. 

**TODO!!** Anything we might want to say about multi-stage builds? E.g. having the scan step as a final stage? 

## Usage limits
Your token will be rate-limited to a reasonable number of scans. Currently this is set to 100 scans per day, though this could change. If you hit rate-limiting issues please do get in touch to discuss your use-case.  

## Community Edition vs Enterprise Edition

The freely-available Community Edition of microscanner scans for vulnerabilities in the image's installed packages. 

Customers of Aqua's commercial Container Security Product have access to additional Enterprise Edition scanning features such as scanning files for vulnerabilities, and scanning for sensitive data included in a container image.  **TODO!!** Check description of Enterprise Edition / commercial version. 

