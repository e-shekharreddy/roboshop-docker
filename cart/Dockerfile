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
# COPY *.js .
# COPY package.json .
# RUN npm install
# ENV REDI_HOST="redis" \
#     CATALOGUE_HOST="catalogue" \
#     CATALOGUE_PORT="8080"
# CMD ["node", "server.js"]