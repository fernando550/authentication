server.js or app.js (entry point for your express server)

1a)
    ***you need to import passport and passport-local to deal with authenticating a user with username and password***
    
    import passport from 'passport';
    import localStrategy from 'passport-local';
    import user from '../models/user'; 
    
    
1b)
    ***Make a user class like this one. It is very important that you use passport-local-mongoose as it gives you a set of methods to register a new user, as well as easily provide authentication functionality to passport so you don't have to***
    
    // ==================== USER MODEL
    var mongoose = require('mongoose');
    var localMongoose = require('passport-local-mongoose');
    
    var userSchema = new mongoose.Schema({
        username: String,
        password: String
    });
    
    userSchema.plugin(localMongoose);
    
    module.exports = mongoose.model('user', userSchema);
        

2a)    
    ***The following is the configuration for your mongoDB, passport and routing***
    
    // ==================== PASSPORT CONFIGURATION
    app.use(require('express-session')({
        secret: 'applepie',
        resave: false,
         saveUninitialized: false
    }));
    
    ***below you will see user.authenticate, user.serializeUser, and user.deserializeUser ***
    ***These functions have been provided by mongoose-local-passport and make your life easy to configure passport***
    app.use(passport.initialize());
    app.use(passport.session());
    passport.use(new localStrategy(user.authenticate()));
    passport.serializeUser(user.serializeUser());
    passport.deserializeUser(user.deserializeUser());
    
    ***you need to set this in order to set up a local variable called user equal your user information*** 
    app.use(function(req, res, next){
        res.locals.user = req.user;
        next();
    });
    
2b)
    ***Make a middleware class like this one. This is what your middleware is going to look like so you can make sure a user is logged in for every step*** 
    
    var midware = {};
    
    ***add different functions called whatever you want ***
    ***this first function added to my exported object is 'isLoggedIn' ***
    
    midware.isLoggedIn = function(req, res, next){
        if(req.isAuthenticated()){
            return next();
        }
        res.redirect('/login');
    };
    
    midware.checkCampgroundOwner = function(req, res, next){
        if (req.isAuthenticated()){
            campground.findById(req.params.id, function(err, aCamp){
                if(err){
                    return res.redirect('back');
                } else {
                    if(aCamp.author.id.equals(req.user._id)){
                        next();
                    } else {
                        return res.redirect('back');
                    }
                }
            });
        } else {
            return res.redirect('back');
        }
    };
    
    midware.checkCommentOwner = function(req, res, next){
        if (req.isAuthenticated()){
            comment.findById(req.params.cid, function(err, aComment){
                if(err){
                    return res.redirect('back');
                } else {
                    if(aComment.author.id.equals(req.user._id)){
                        next();
                    } else {
                        return res.redirect('back');
                    }
                }
            });
        } else {
            return res.redirect('back');
        }
    };
    
    module.exports = midware;
    
3a)
    *** Lastly, you're going to need some routes to deal with logging in, as well as logging out. You're going to want to have a navbar that keeps your session available by checking the user object. If it's empty then it wont show you your log in name in the navbar and it'll just say 'sign in'***
    
    *** I only have the header in EJS format, but you get the point***
    
    <nav class="navbar navbar-default">
        <div class="container-fluid">
            <div class="navbar-header">
                <a class="navbar-brand" href="/">YelpCamp</a>
            </div>
            
            <div class="collapse navbar-collapse">
                <ul class="nav navbar-nav navbar-right">
                    <% if(!user){ %>
                        <li><a href="/login">Login</a></li>
                        <li><a href="/register">Register</a></li>
                    <% } else { %>
                        <li><a href='#'>Signed In As <%= user.username %></a></li>
                        <li><a href="/logout">LogOut</a></li>
                    <% } %>
                </ul>
            </div>
        </div>
    </nav>
    
    
    ***and this is what the 'index' or generic routes should look like***
    
    var express = require('express'),
    router  = express.Router(),
    user = require('../models/user'),
    passport = require('passport');

    // START
    router.get('/', function(req, res){
        res.render('home');
    });
    
    // ==================== AUTHENTICATION ROUTES
    
    // REGISTER - account creation page
    router.get('/register', function(req, res){
        res.render('register');
    });
    
    // REGISTER - create account
    router.post('/register', function(req, res){
        user.register(new user({username: req.body.username}), req.body.password, function(err, user){
            if(err){
                console.log(err);
                return res.render('register');
            }
            passport.authenticate('local')(req, res, function(){
                res.redirect('/home');
            });
        });
    });
    
    // LOGIN - account login page
    router.get('/login', function(req, res){
        res.render('login');
    });
    
    // LOGIN - sign in
    router.post('/login', passport.authenticate('local', {
            successRedirect: '/home',
            failureRedirect: '/login'
        }), function(req, res){
    });
    
    // LOGOUT
    router.get('/logout', function(req, res){
        req.logout();
        res.redirect('/home');
    });
    
    
    module.exports = router