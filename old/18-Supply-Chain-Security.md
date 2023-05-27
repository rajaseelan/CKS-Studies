# Supply Chain Security - Image Footprint

## Image footprint

* Docker containers
* Reducing Image Footprint (multi stage)
* Secure Images

## Layers in Docker Containers
The following Dockerfile instructions create layers:  
* RUN
* COPY
* ADD

## Reducing Image Footprint
<details>
<summary>How to reduce image size / footprint</summary>

Multi Stage Builds.  
Install all Dependencies and build.  
Copy results to a new base minimum base container.

</details>

---

## Lab: Reducing Image Footprint

<details>
<summary>Sample Dockerfile with multistage build</summary>

```Dockerfile
FROM ubuntu
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y golang-go
COPY app.go .
RUN CGO_ENABLED=0 go build app.go

FROM alpine
COPY --from=0 /app . 
CMD["./app"]
```
</details>

---

## Lab: Securing and Hardening Images

<details>
<summary>Best Practices for securing Images</summary>

1. Use Tags for images, pull specific versions

1. Don't run as root
i.e 

    # Create a user  
RUN addgroup -S appgroup && adduser -S appuser -G appgroup -h /home/appuser
    # Copy app into user home
COPY --from=0 /app /home/appuser
    # All commands from below run as user appuser  
USER appuser
CMD [ "/home/appuser/app" ]
    # make docker filesystem readonly  
RUN chmod a-w /etc
    # remove shell access  
RUN rm -rf /bin/*

</details>
 
## Reading:
* Dockerfile Best Practices


