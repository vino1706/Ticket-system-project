# Stage 1: Build the production static files
FROM node:20-alpine AS build
WORKDIR /app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the frontend source code and compile the application
COPY . .
RUN npx vite build

# Stage 2: Serve the compiled static assets using Nginx
FROM nginx:1.25-alpine
WORKDIR /usr/share/nginx/html

# Clean the default Nginx html directory
RUN rm -rf ./*

# Copy the compiled production build files from Stage 1 into Nginx
# Note: Vite outputs its production build into the 'dist' directory
COPY --from=build /app/dist .

# Copy a custom Nginx config to handle React Single Page Application (SPA) routing
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
