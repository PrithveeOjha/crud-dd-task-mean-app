# Stage 1: Use a lean Node.js image
FROM node:18-alpine
WORKDIR /usr/src/app

# Copy package.json and install Node dependencies
COPY package*.json ./
RUN npm install

# Bundle application source
COPY . .

# Backend runs on port 8080
EXPOSE 8080

CMD [ "node", "server.js" ]
