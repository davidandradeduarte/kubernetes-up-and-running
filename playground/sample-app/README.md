# Build and run sample app

## Build the Docker image
```bash
docker build -t sample-app .
```

## Run a container from the Docker image
```bash
docker run --rm -p 3000:3000 sample-app
```