# Let's setup a login session thing

The following code is what worked for your *iron_yard/timetracker* app.

## Gems
Enable Bcrypt by uncommenting it in the Gemfile.

## Migrations
When setting up your user migration be sure to use password_digest:string for the password field.

## Routes

    root   'projects#index'

Directs to a dashboard page.  Any old page will do, so long as it's what you want people to see when they are logged in and pointing at root.

    get    'login' => 'sessions#new'
    post   'login' => 'sessions#create'

These will direct the user to the login page, either through redirects or by pointing at root/login from the address bar.

    delete 'logout' => 'sessions#destroy'

When directed with the delete method or logout_path, the session will be destroyed.

    resources :developers
    resources :time_entries
    resources :projects

All of the rest of the crap you need for the app to work.

## Controllers

For basic login/logout you will only need to modify two controllers:

### application_controller.rb

    before_action :require_login  

Requires that the following method is called to verify whether someone has logged in prior to loading any page.

    def logged_in?
      session[:user_id] != nil
    end

Simply checks if a session has been created.

    def require_login
      unless logged_in?
        redirect_to login_path, notice: 'Please log in to view content'
      end
    end

Redirects back to login page if a session hasn't been created.

### sessions_controller.rb

    skip_before_action :require_login, only: [:create, :new]

Skips the login check from application_controller for new and create so that the login page doesn't fall into a redirect loop.

    def new
    end

    def create
      dev = Developer.find_by_email(params[:email])
      if dev && dev.authenticate(params[:password])
        session[:user_id] = dev.id
        redirect_to root_path, notice: "You have been successfully logged in."
      else
        flash.now[:alert] = 'Invalid email/password combination'
        render :new
      end
    end

Looks for a Developer (User) by the email address supplied at the login screen, then if it finds one and can authenicate the password it will populate the session with any variables you want to store there and redirects to the desired view.  Otherwise it redirects back to the login page with an error message.

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

Form is in the sessions/new.html.erb and is setup to point to the login_path, which will pass email and password to sessions#create.

### _nav.html.erb
Or just make sure it's on every page so long as the user is logged in.

    <%= link_to "LOGOUT", logout_path, method: :delete %>

Points to the destroy path in your controller.  This relies heavily on how routes are setup.
