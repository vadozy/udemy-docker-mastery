FROM ubuntu

RUN apt-get update                                      && \
    apt-get install -y --no-install-recommends             \
        iputils-ping                                       \
        cowsay                                             \
        screenfetch                                     && \
    rm -rf /var/lib/apt/lists/*

# CMD ["ping", "localhost"]
CMD ["cowsay", "Yo, CMD !!"]