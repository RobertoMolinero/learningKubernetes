# Complex Application

For a modern web application the different components are booted as separate containers.

```
docker run -d --name=redis redis
docker run -d --name=db postgresql:9.4
docker run -d --name=vote -p 5000:80 voting-app
docker run -d --name=result -p 5001:80 result-app
docker run -d --name=worker worker
```

All components function perfectly. However, they do not know about each other and cannot communicate with each other.

To get a stack of connected components, the links must be specified with the '--link' option.

```
docker run -d --name=redis redis
docker run -d --name=db postgresql:9.4
docker run -d --name=vote -p 5000:80 --link redis:redis voting-app
docker run -d --name=result -p 5001:80 --link db:db result-app
docker run -d --name=worker --link redis:redis --link db:db worker
```