<h1> MEAN Stack on AWS Ubuntu </h1>

<h3> This repository provides the necessary steps and code for deploying a MEAN (MongoDB, Express.js, Angular, Node.js) stack application on an Ubuntu server running on Amazon Web Services (AWS) EC2 instance. The deployment follows a DevOps approach, ensuring automation and ease of setup.
Prerequisites </h3>


Update and upgrade the system packages:

        sudo apt update
        sudo apt upgrade

Install the necessary packages for managing certificates:

    sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

Install Node.js:

    curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    sudo apt install -y nodejs


Install MongoDB:

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 0C49F3730359A14518585931BC711F9BA15703C6
    echo "deb [ arch=amd64 ] https://repo.mongodb.org/apt/ubuntu trusty/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
    sudo apt install -y mongodb
    sudo service mongodb start
    sudo systemctl status mongodb

Install NPM and the required dependencies:
    
    sudo apt install -y npm
    sudo npm install body-parser
    sudo npm install express mongoose

Create a directory for your project and navigate into it:

    mkdir Books && cd Books


Create the server.js file and add the following code:

    vim server.js


    var express = require('express');
    var bodyParser = require('body-parser');
    var app = express();
    app.use(express.static(__dirname + '/public'));
    app.use(bodyParser.json());
    require('./apps/routes')(app);
    app.set('port', 3300);
    app.listen(app.get('port'), function() {
        console.log('Server up: http://localhost:' + app.get('port'));
    });

Create a directory named apps and navigate into it:

    mkdir apps && cd apps


Create the routes.js file and add the following code:

    vi routes.js


    var Book = require('./models/book');
    module.exports = function(app) {
      app.get('/book', function(req, res) {
        Book.find({}, function(err, result) {
          if (err) throw err;
          res.json(result);
        });
      });
      app.post('/book', function(req, res) {
        var book = new Book({
          name: req.body.name,
          isbn: req.body.isbn,
          author: req.body.author,
          pages: req.body.pages
        });
        book.save(function(err, result) {
          if (err) throw err;
          res.json({
            message: "Successfully added book",
            book: result
          });
        });
      });
      app.delete("/book/:isbn", function(req, res) {
        Book.findOneAndRemove(req.query, function(err, result) {
          if (err) throw err;
          res.json({
            message: "Successfully deleted the book",
            book: result
          });
        });
      });
      var path = require('path');
      app.get('*', function(req, res) {
        res.sendFile(path.join(__dirname + '/public', 'index.html'));
      });
    };


Create a directory named models and navigate into it:

    mkdir models && cd models


Create the script.js file and add the following code:

    vi script.js
    
    var app = angular.module('myApp', []);
    app.controller('myCtrl', function($scope, $http) {
      $http({
        method: 'GET',
        url: '/book'
      }).then(function successCallback(response) {
        $scope.books = response.data;
      }, function errorCallback(response) {
        console.log('Error: ' + response);
      });
      $scope.del_book = function(book) {
        $http({
          method: 'DELETE',
          url: '/book/:isbn',
          params: { 'isbn': book.isbn }
        }).then(function successCallback(response) {
          console.log(response);
        }, function errorCallback(response) {
          console.log('Error: ' + response);
        });
      };
      $scope.add_book = function() {
        var body = '{ "name": "' + $scope.Name +
          '", "isbn": "' + $scope.Isbn +
          '", "author": "' + $scope.Author +
          '", "pages": "' + $scope.Pages + '" }';
        $http({
          method: 'POST',
          url: '/book',
          data: body
        }).then(function successCallback(response) {
          console.log(response);
        }, function errorCallback(response) {
          console.log('Error: ' + response);
        });
      };
    });
    


Create the index.html file and add the following code:

vi index.html

    <!doctype html>
    <html ng-app="myApp" ng-controller="myCtrl">
      <head>
        <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
        <script src="script.js"></script>
      </head>
      <body>
        <div>
          <table>
            <tr>
              <td>Name:</td>
              <td><input type="text" ng-model="Name"></td>
            </tr>
            <tr>
              <td>Isbn:</td>
              <td><input type="text" ng-model="Isbn"></td>
            </tr>
            <tr>
              <td>Author:</td>
              <td><input type="text" ng-model="Author"></td>
            </tr>
            <tr>
              <td>Pages:</td>
              <td><input type="number" ng-model="Pages"></td>
            </tr>
          </table>
          <button ng-click="add_book()">Add</button>
        </div>
        <hr>
        <div>
          <table>
            <tr>
              <th>Name</th>
              <th>Isbn</th>
              <th>Author</th>
              <th>Pages</th>
            </tr>
            <tr ng-repeat="book in books">
              <td>{{book.name}}</td>
              <td>{{book.isbn}}</td>
              <td>{{book.author}}</td>
              <td>{{book.pages}}</td>
              <td><input type="button" value="Delete" data-ng-click="del_book(book)"></td>
            </tr>
          </table>
        </div>
      </body>
    </html>



Navigate back to the project's root directory:

    cd ..


Test the application locally:

    curl -s http://localhost:3300


Access the application using your EC2 instance's public IP address or domain name followed by port 3300 (e.g., http://ec2-public-ip:3300).
