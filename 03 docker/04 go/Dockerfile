# Specify a base image
FROM golang:1.13

WORKDIR /go/src/app

COPY heyHoLetsGo.go .

RUN go get -d -v ./...
RUN go install -v ./...

# Default command
CMD ["app"]










