# Start from a Node.js 16 (LTS) image
FROM node:16
# Specify the directory inside the image in which all commands will run
WORKDIR /usr/src/app
# Copy package files and install dependencies
COPY package*.json ./
RUN npm install
RUN npm install express
# Copy all of the app files into the image
COPY . .
# The default command to run when starting the container
CMD [ "npm", "start" ]