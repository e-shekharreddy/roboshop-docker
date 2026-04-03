FROM node:20
WORKDIR /app
COPY *.js .
COPY package.json .
RUN npm install
ENV REDI_HOST="redis" \
    CATALOGUE_HOST="catalogue" \
    CATALOGUE_PORT="8080"
CMD ["node", "server.js"]