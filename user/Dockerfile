FROM node:20.20.2-alpine3.23 AS builder
WORKDIR /app
COPY package.json .
COPY *.js .
RUN npm install

FROM node:20.20.2-alpine3.23
WORKDIR /app
EXPOSE 8080
COPY --from=builder /app /app
# this ENV is handling in k8 via configMAp
# ENV REDI_HOST="redis" \
#     CATALOGUE_HOST="catalogue" \
#     CATALOGUE_PORT="8080"
RUN addgroup -S roboshop && adduser -S roboshop -G roboshop
RUN chown -R roboshop:roboshop /app
USER roboshop
CMD ["server.js"] 
ENTRYPOINT ["node"] 

# FROM node:20
# WORKDIR /app 
# # docker layers we used so we bring fixed one to top like ENV and COPY which those are not change frequently
# ENV REDIS="true" \
#     REDIS_URL="redis://redis:6379" \
#     MONGO="true" \
#     MONGO_URL="mongodb://mongodb:27017/users"
# EXPOSE 8080
# COPY package.json .
# COPY *.js .
# RUN npm install
# CMD ["node", "server.js"]