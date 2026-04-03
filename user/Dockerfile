FROM node:20
WORKDIR /app
COPY *.js .
COPY package.json .
RUN npm install
ENV REDIS="true" \
    REDIS_URL="redis://redis:6379" \
    MONGO="true" \
    MONGO_URL="mongodb://mongodb:27017/users"
CMD ["node", "server.js"]