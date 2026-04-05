FROM node:20
WORKDIR /app 
# docker layers we used so we bring fixed one to top like ENV and COPY which those are not change frequently
ENV REDIS="true" \
    REDIS_URL="redis://redis:6379" \
    MONGO="true" \
    MONGO_URL="mongodb://mongodb:27017/users"
EXPOSE 8080
COPY package.json .
COPY *.js .
RUN npm install
CMD ["node", "server.js"]