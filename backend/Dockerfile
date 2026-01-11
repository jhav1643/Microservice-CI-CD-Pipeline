FROM node:16-alpine

WORKDIR /app

COPY package*.json ./

# Install ALL dependencies including devDependencies
ENV NODE_ENV=development
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]
