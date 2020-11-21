# docker-lamp-stack

Code / notes written while following [How to Create PHP Development Environments with Docker Compose](https://www.youtube.com/watch?v=l0jb-N5H52A)

## Notes

### What is Docker Compose [(5:05)](https://youtu.be/l0jb-N5H52A?t=305)

- allows running of multiple containers applications
  - defined in `docker-compose.yaml` file

### Using Docker Compose [(6:42)](https://youtu.be/l0jb-N5H52A?t=402)

> `docker-compose [command]`

- Controlling the environment
  - `up` - sets up environment (-b flag to run in background)
  - `down` - tears down (removes) environment
  - `stop` - pause (temporary)
  - `start` - resume
- Monitoring and Troubleshooting
  - `ps` - overview (status) of all containers
  - `logs` - show logs (d'uh)
  - `top` - show active processes
  - `kill` - kill containers
- Executing commands on containers
  - `exec <service_name> [command]`

### Example Nginx Service [(5:45)](https://youtu.be/l0jb-N5H52A?t=345)

```yaml
version: "3.7"    # docker-compose version number
services:
  web:            # name of the service (user defined)
    image: nginx  # name of the image (find on dockerhub)
    ports:        # defines port-forwarding
      - "8080:80" # redirect all requests on host port 8080 to container port 80
```
