# Let's setup a login session thing

The following code is what worked for your *iron_yard/timetracker* app.

## Gems
Enable Bcrypt by uncommenting it in the Gemfile.

## Migrations
When setting up your `User` migration be sure to use `password_digest:string` for the password field.

## Routes

    root   'projects#index'

Directs to a dashboard page.  Any old page will do, so long as it's what you want people to see when they are logged in and pointing at root.  This becomes useful when you want Users to load a particular view after logging in and any time they punch in the root address.

    get    'login' => 'sessions#new'
    post   'login' => 'sessions#create'

These will direct the user to the login page, either through redirects or by pointing at root/login from the address bar.  Remember that your login form with be on the `sessions/new` view.

    delete 'logout' => 'sessions#destroy'

When directed with `method: :delete`  or `logout_path`, the session will be destroyed.  You'll see how below in the controllers.

## Controllers

For basic login/logout you will only need to modify two controllers:

### application_controller.rb

    before_action :require_login  

Requires that the following method is called to verify whether someone has logged in prior to loading any page.

    def require_login
      unless logged_in?
        redirect_to login_path, notice: 'Please log in to view content'
      end
    end

Redirects back to login page if a session hasn't been created.  What is `logged_in?`, you ask?

    def logged_in?
      @current_user ||= User.find_by_id(session[:user_id])
    end

Simply checks if a session has been created.  If a User is found it will store it in an instance variable.  As you move around the site it will require you to check `logged_in?` at each page load.  We don't want it to query the database everytime so we use memoization  (that `||=` thing) to tell the app that if we've already got something in `@current_user`, just skip the query.



### sessions_controller.rb

    skip_before_action :require_login

Skips the login check from application_controller so that the login page doesn't fall into a redirect loop.

    def new
    end

    def create
      dev = User.find_by_email(params[:email])
      if user && user.authenticate(params[:password])
        session[:user_id] = user.id
        redirect_to root_path, notice: "You have been successfully logged in."
      else
        flash.now[:alert] = 'Invalid email/password combination'
        render :new
      end
    end

Looks for a User by the email address supplied at the login screen, then if it finds one and can authenicate the password it will populate the session with any variables you want to store there and redirects to the desired view.  Here, I've only put the User's ID into the session since I can look up his other information based on this later if needed.  Otherwise it redirects back to the login page with an error message.

    def destroy
      session[:user_id] = nil
      redirect_to login_path, notice: 'Logout successful.'
    end

When the logout link is clicked it should point here.  When called it should destroy the session variables and redirect to the login page.

## Models

One line in one model needs to be added so that passwords don't get mucked up.

### models/developer.rb

    has_secure_password

Require that the User has a password before creating a record.

## Views

If you want to log in and out you need to supply tools for the user to use for that.

### sessions/new

    <%= form_tag login_path do %>
    <p>
      <%= label_tag :email %>
      <%= email_field_tag :email  %>
    </p>
    <p>
      <%= label_tag :password %>
      <%= password_field_tag :password %>
    </p>
    <p>
      <%= submit_tag "Login" %>
    </p>
    <% end %>

Form is in `sessions/new.html.erb` and is setup to point to the login_path, which will pass email and password to sessions#create.  This can also be stored in a partial so that you can also display it elsewhere in your app if you like.

### _nav.html.erb
Or just make sure this next bit is on every page so long as the user is logged in.  I'm just fond of putting all links into a navigation partial so that it's easy to render on every page.  Feel free to put this link_to wherever you like, though.  I won't be mad.  I Promise.

    <%= link_to "LOGOUT", logout_path, method: :delete %>

Points to the destroy path in your controller.  This relies heavily on how routes are setup.
