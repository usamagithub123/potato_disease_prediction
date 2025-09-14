# Build stage
FROM node:20-alpine AS build
WORKDIR /app
COPY package*.json ./
# Use legacy OpenSSL if needed
ENV NODE_OPTIONS=--openssl-legacy-provider
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

