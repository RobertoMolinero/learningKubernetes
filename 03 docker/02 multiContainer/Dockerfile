# Specify a base image
FROM node:alpine

WORKDIR '/app'

# Copy the npm dependency file
COPY package.json .

# Install some dependencies
RUN npm install

# Copy the project files
COPY . .

# Default command
CMD ["npm", "start"]