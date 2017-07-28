server.js or app.js (entry point for your express server)

1a)
    <!--you need to import passport and passport-local to deal with authenticating a user with username and password-->
    import passport from 'passport';
    import localStrategy from 'passport-local';
    import user from '../models/user'; <!-- important to import your user class -->
    
1b)
        (
            <!--Make a user class like this one -->
            <!--It is very important that you use passport-local-mongoose as it gives you a set of methods to register a new user,-->
            <!--as well as easily provide authentication functionality to passport so you don't have to.-->
            
            var mongoose = require('mongoose');
            var localMongoose = require('passport-local-mongoose');
            
            var userSchema = new mongoose.Schema({
                username: String,
                password: String
            });
            
            userSchema.plugin(localMongoose);
            
            module.exports = mongoose.model('user', userSchema);
        )

2a)    
    The following is the configuration for your mongoDB, passport and routing
    
    // ==================== PASSPORT CONFIGURATION
    app.use(require('express-session')({
        secret: 'applepie',
        resave: false,
         saveUninitialized: false
    }));
    
    <!-- below you will see user.authenticate, user.serializeUser, and user.deserializeUser -->
    <!-- These functions have been provided by mongoose-local-passport and make your life easy to configure passport -->
    app.use(passport.initialize());
    app.use(passport.session());
    passport.use(new localStrategy(user.authenticate()));
    passport.serializeUser(user.serializeUser());
    passport.deserializeUser(user.deserializeUser());
    
    <!-- you need to set this in order to set up a local variable called user equal your user information -->
    app.use(function(req, res, next){
        res.locals.user = req.user;
        next();
    });
    
2b)
        (
            <!-- Make a middleware class like this one -->
            <!-- This is what your middleware is going to look like so you can make sure a user is logged in for every step -->
            var midware = {};
            
            <!-- add different functions called whatever you want -->
            <!-- this first function added to my exported object is 'isLoggedIn' -->
            midware.isLoggedIn = function(req, res, next){
                if(req.isAuthenticated()){
                    return next();
                }
                <!--  req.flash('error', 'You must be logged in to continue');  I haven't configured flashes yet -->
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
        )