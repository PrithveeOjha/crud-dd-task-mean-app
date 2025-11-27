# Stage 1: Build the Angular application
FROM node:18-alpine as build
WORKDIR /app

# Copy package.json and install dependencies
COPY package.json package-lock.json ./
RUN npm install

# Copy source code and build the app
COPY . .
# The Angular build output path is dist/angular-15-crud (from angular.json)
RUN npm run build -- --configuration production

# Stage 2: Create a lightweight Nginx serving image
FROM nginx:alpine

# Copy the custom Nginx config (created in Step 3)
COPY ./nginx/default.conf /etc/nginx/conf.d/default.conf

# Copy the built Angular app from the 'build' stage to the Nginx public directory
# The built app is in /app/dist/angular-15-crud
COPY --from=build /app/dist/angular-15-crud /usr/share/nginx/html

# Expose port 80, which Nginx uses by default
EXPOSE 80

# Command to run Nginx remains the default one
CMD ["nginx", "-g", "daemon off;"]
