# Start from a Node.js 14 image
FROM node:14
# Specify the directory inside the image in which all commands will run
WORKDIR /usr/src/app
# Copy package files and install dependencies
COPY package*.json app.js ./
RUN npm install
# Copy all of the app files into the image
COPY . .
# The default command to run when starting the container
CMD ["node", "app.js"]