# Specify a base image
FROM node:alpine

# Set a working directory
WORKDIR /usr/app

# Copy the npm dependency file
COPY ./package.json ./

# Install some dependencies
RUN npm install

# Copy the project files
COPY ./ ./

# Default command
CMD ["npm", "start"]