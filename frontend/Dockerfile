FROM node:16 AS build

# Set the working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# creating the build time variable: --build-arg REACT_APP_BACKEND_URL=""
ARG REACT_APP_BACKEND_URL=""
ENV REACT_APP_BACKEND_URL=${REACT_APP_BACKEND_URL}

# Install the dependicies
RUN npm install 

COPY . .

# Create the static React Build
RUN npm run build

# Final Stage
FROM nginx:alpine

COPY --from=build /app/build/ /usr/share/nginx/html

EXPOSE 80

# since react-router is not used so, custom nginx.conf file is not needed
CMD ["nginx", "-g", "daemon off;"]
