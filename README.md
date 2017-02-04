##Rails API and `has_many_through` example

**Deployed Front-end on GH Pages**: https://marcwright.github.io/rails5-api-angular-wdir2-demo-frontend/

**Front-end GitHub repo**: https://github.com/marenwoodruff/rails_api_front_end

**Deployed Backend API**: https://rails5-api-wdir2-demo-backend.herokuapp.com/api/patients

> Note: other endpoints are `/api/doctors` and `/api/appointments`

**Back-end GitHub Repo**: https://github.com/marenwoodruff/rails5_api_with_angular

We're gonna build a Rails API app to demo this relationship: http://guides.rubyonrails.org/association_basics.html#the-has-many-through-association

- What is an API?
- Why would you use one?
    

NOTE: Be sure to create this app outside your WDIR class folder. This app will need it's own `.git` since we're pushing to Heroku.

1. `rails new rails5_api_demo --api -d=postgresql -T` 
    - --api is the key, it will create a rails app without views
    - also make sure to add _api to your app name
2. `cd` && `subl .`
3. Add the following lines to config/env/development.rb, around line 44
    - The config folder stores your app configurations for different environments- most is done in app.rb
    - This will improve the error responses you receive to JSON requests
    - Will render an html page with debugging information
    - You could use :api instead of default to preserve the response format

    ```rb
    # Add Rails 4.2 serverside rendered errors
    config.debug_exception_response_format = :default
    ```
    
3. In your `Gemfile` uncomment out `gem 'rack-cors'` around line 26. 
4. In your `Gemfile` add `gem 'faker'`. We'll use this later on to seed our database in both localhost and production.
    - Read more about the faker gem at: https://github.com/stympy/faker
    - Where would you go to find the latest version of Ruby gems?
5. `bundle install`
4. Inside of `config/initializers/cors.rb`, add the following:

    ```rb
    Rails.application.config.middleware.insert_before 0, Rack::Cors do
      allow do
        origins '*'
    
        resource '*',
          headers: :any,
          methods: [:get, :post, :put, :patch, :delete, :options, :head]
      end
    end
    ```

    > We're pretty much uncommenting out the code and changing `origins` to `'*'`
    - you don't want to do this in a prod app, but it will be fine for our purposes today

6. `rails db:create db:migrate`

8. Doctor - `rails g model Doctor name specialty insurance:boolean gender`
    - Why don't I have to add the type behind some of these fields?
    - What are the different field types that are available?

    ```rb
    class Doctor < ApplicationRecord
      has_many :appointments, dependent: :destroy
      has_many :patients, through: :appointments
      validates :name, presence: true, uniqueness: true
    end
```
9. Patient - `rails g model Patient name insurance_co gender new_patient:boolean`

    ```rb
    class Patient < ApplicationRecord
      has_many :appointments
      has_many :doctors, through: :appointments
      validates :name, presence: true, uniqueness: true
    end
    ```
7. Appointment - `rails g model Appointment location day reason doctor:references patient:references`

    ```rb
    class Appointment < ApplicationRecord
      belongs_to :doctor
      belongs_to :patient
    end
```
> Rails added the `belongs_to` since we used `:references`

10. `rails db:migrate`

## Explore the associations
11. In a new tab, enter `rails c`
5. If you have `rails c` issues: 
    - `gem install rb-readline`
    -  add `gem 'rb-readline', '~> 0.5.3'` to `Gemfile`

### Create instances of a Patient, Doctor, Appointment 
12. `patient = Patient.create(name: "Schmitty", insurance_co: "Anthem", gender: "M", new_patient: true)`
13. `doctor = Doctor.create(name: "Dr. Robert", specialty: "feet", insurance: true, gender: "M")`
14. `appointment_one = Appointment.create(location: "Grady General", day: "Monday", reason: "smelly feet", doctor_id: 1, patient_id: 1)`
15. `doctor.patients` or `patient.doctors`
16. `patient.appointments` or `doctor.appointments`

> We can now access a Doctor's Patients and vice-versa through Appointments

### Create new Patients and Doctors through Appointments
17. `patient1 = doctor.patients.create(name: "Diesel", insurance_co: "Blue Cross", gender: "M", new_patient: false)`
18. `doctor1 = patient.doctors.create(name: "Dr. Foster", specialty: "Veterinary", gender: "M")`
19. `patient1.appointments.create(APPOINMENT ATTRIBUTE DETAILS HERE)` or `doctor1.appointments.create(APPOINMENT ATTRIBUTE DETAILS HERE)`
19. `appointment_two = Appointment.create(location: "Inman Park", day: "Friday", reason: "shots", doctor_id: doctor.id, patient_id: patient.id)`

## Doctor CRUD
20. `rails g controller doctors`

    ```rb
    class DoctorsController < ApplicationController
      # GET /doctors
      def index
        doctors = Doctor.all
    
        render json: doctors
      end
    
      # GET /doctors/1
      def show
        doctor = Doctor.find(params[:id])
        render json: doctor
      end
    
      # POST /doctors
      def create
        doctor = Doctor.new(doctor_params)
        puts(doctor_params)
    
        if doctor.save
          render json: doctor, status: :created, location: doctor
        else
          render json: doctor.errors, status: :unprocessable_entity
        end
      end
    
      # PATCH/PUT /doctors/1
      def update
        doctor = Doctor.find(params[:id])
        if doctor.update(doctor_params)
          render json: doctor
        else
          render json: doctor.errors, status: :unprocessable_entity
        end
      end
    
      # DELETE /doctors/1
      def destroy
        doctor = Doctor.find(params[:id])
        doctor.destroy
    
        render json: {status: 204}
      end
    
      private
        # Only allow a trusted parameter "white list" through.
        def doctor_params
          params.require(:doctor).permit(:name, :specialty, :insurance, :gender)
        end
    end
```

1. `config/routes.rb`

    ```rb
    Rails.application.routes.draw do
        resources :doctors, path: '/api/doctors'
        # For details on the DSL available within this file, see http://guides.rubyonrails.org/routing.html
    end
```

1. `rails s`
2. Postman - Create/Post

![](https://i.imgur.com/GE4DPV0.png)

    > Make sure that `raw` is set to `JSON(application/json)`
1. Postman - Update/Put

    ![](https://i.imgur.com/HNAJJmw.png)

1. Postman - Delete/Delete
2. Postman - Read/Index-Show

## Patient CRUD
3. `rails g controller patients`

    ```rb
    class PatientsController < ApplicationController
      # GET /patients
      def index
        patients = Patient.all
    
        render json: patients
      end
    
      # GET /patients/1
      def show
        patient = Patient.find(params[:id])
        render json: patient
      end
    
      # POST /patients
      def create
        patient = Patient.new(patient_params)
        puts(patient_params)
    
        if patient.save
          render json: patient, status: :created, location: patient
        else
          render json: patient.errors, status: :unprocessable_entity
        end
      end
    
      # PATCH/PUT /patients/1
      def update
        patient = Patient.find(params[:id])
        if patient.update(patient_params)
          render json: patient
        else
          render json: patient.errors, status: :unprocessable_entity
        end
      end
    
      # DELETE /patients/1
      def destroy
        patient = Patient.find(params[:id])
        patient.destroy
        render json: {status: 204}
      end
    
      private
        # Only allow a trusted parameter "white list" through.
        def patient_params
          params.require(:patient).permit(:name, :new_patient, :insurance_co, :gender)
        end
    end
```

1. `routes.rb` - `resources :patients, path: '/api/patients'`

## Appointment CRUD

1. `rails g controller appointments`
    
    ```ruby
    class AppointmentsController < ApplicationController

      # An optional method for when we wire up Angular
      def get_all
        doctors = Doctor.all
        appointments = Appointment.all
        patients = Patient.all
        all_data = {patients: patients, doctors: doctors, appointments: appointments}
        render json: all_data
      end
    
      # GET /appointments
      def index
        appointments = Appointment.all
    
        render json: appointments
      end
    
      # GET /appointments/1
      def show
        appointment = Appointment.find(params[:id])
        render json: appointment
      end
    
      # POST /appointments
      def create
        appointment = Appointment.new(appointment_params)
        puts(appointment_params)
    
        if appointment.save
          render json: appointment, status: :created, location: appointment
        else
          render json: appointment.errors, status: :unprocessable_entity
        end
      end
    
      # PATCH/PUT /appointments/1
      def update
        appointment = Appointment.find(params[:id])
        if appointment.update(appointment_params)
          render json: appointment
        else
          render json: appointment.errors, status: :unprocessable_entity
        end
      end
    
      # DELETE /appointments/1
      def destroy
        appointment = Appointment.find(params[:id])
        appointment.destroy
        render json: {status: 204}
      end
    
      private
    
        # Only allow a trusted parameter "white list" through.
        def appointment_params
          params.require(:appointment).permit(:location, :day, :reason, :doctor_id, :patient_id)
        end
    end
    ```


2. `routes.rb` - `resources :appointments, path: '/api/appointments'`
3. Postman - Create

    ![](https://i.imgur.com/0GRG0xA.png)
    


## Heroku

1. `git init`, `git add -A`, `git commit -m "First Commit"`
1. `$ heroku create`
2. `$ git push heroku master`
2. `$ heroku run rails db:migrate`

#### Add data through the Heroku Rails Console

3. `$ heroku run console`

    `patient = Patient.create(name: "Schmitty", insurance_co: "Anthem", gender: "M", new_patient: true)`
`doctor = Doctor.create(name: "Dr. Robert", specialty: "feet", insurance: true, gender: "M")`
`appointment_one = Appointment.create(location: "Grady General", day: "Monday", reason: "smelly feet", doctor_id: 1, patient_id: 1)`
`doctor.patients.create(name: "Diesel", insurance_co: "Blue Cross", gender: "M", new_patient: false)`
`patient.doctors.create(name: "Dr. Foster", specialty: "Veterinary", gender: "M")`
`appointment_two = Appointment.create(location: "Inman Park", day: "Friday", reason: "shots", doctor_id: doctor.id, patient_id: patient.id)`

#### We could also add some data to our `db/seeds.rb` using the Faker gem

1. Add to `db/seeds.rb`

    ```rb
Appointment.destroy_all
Patient.destroy_all
Doctor.destroy_all

    10.times do
        Patient.create(name: Faker::Name.name, insurance_co: Faker::Beer.malts, gender: "F", new_patient: Faker::Boolean.boolean)
        Doctor.create(name: Faker::Name.name, specialty: Faker::Hipster.word, gender: "F", insurance: Faker::Boolean.boolean)
    end

    10.times do
        Appointment.create(location: Faker::University.name, day: Faker::Date.forward(23), reason: Faker::Hipster.sentence(6), patient_id: Faker::Number.between(1, 10), doctor_id: Faker::Number.between(1, 10))
    end
```
1. `rails db:reset`
    > Note: This will drop the local databases, recreate, migrate and seed 

1. Make sure to do another `git add -A` and `git commit` to push the file up to Heroku so we can seed our production database.
2. `$ heroku pg:reset DATABASE`
3. `$ heroku run rails db:migrate`
1. Then run `$ heroku run rails db:seed`

### Test out the API with Postman

1. `<NAME_ OF_YOUR_HEROKU_APP_HERE>/api/patients`
2. `<NAME_ OF_YOUR_HEROKU_APP_HERE>/api/doctors`
3. Try to add a new Patient.

    ![](https://i.imgur.com/4kSSM74.png)
    
1. Try to add an Appointment.

    ![](https://i.imgur.com/4Njvh1t.png)  


    
## Angular

1. `cd ..1` && `mkdir rails5-api-angular-frontend_2` && `cd` into it
2. `npm init -y`
    - will fill in all of the angular questions
4. `touch server.js`

    ```js
    var express = require('express');
    var app     = express();
    var path    = require('path');

    app.use(express.static(path.join(__dirname,'public')));

    app.get('/', function(req, res){
        res.render('index');
    });

    app.listen(4000, function(){
        console.log("app listening on port 4000");
    });
    ```
    
1. `npm install --save express path`
2. `mkdir public`
1. `mkdir public/js`
1. `touch public/js/app.js`

    ```js
    (function(){
        angular.module('Rails5', []);
    })()
```

1. `touch public/js/rails5Controller.js`

    ```js
    (function(){
      angular.module('Rails5')
        .controller('rails5Controller', rails5Controller);
    
    
      function rails5Controller($http){
    
        var self = this;        
        var server = "<YOUR_HEROKU_API_LINK_HERE>"
        // For example, var server = 'https://enigmatic-garden-65625.herokuapp.com/api/'
    
        self.doctors = [];
    
        $http.get(`${server}/doctors`)
          .then(function(response) {
              console.log(self.doctors[0].name);
              return self.doctors = response.data;
        });
      }
    })()
``` 

1. `touch public/index.html`
    - to see the doctors on the index.html page

    ```html
    <!DOCTYPE html>
    <html ng-app="Rails5">
      <head>
        <meta charset="utf-8">
        <title>Rails 5 API App</title>
        <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.15/angular.min.js"></script>
        <script src='js/app.js'></script>
        <script src='js/rails5Controller.js'></script>
      </head>
      <body ng-controller="rails5Controller as rails">
        <h1>Doctors</h1>
        <ul ng-repeat="doctor in rails.doctors">
            <li>{{doctor.name}}</li>
            <li>{{doctor.specialty}}</li>
            <li>{{doctor.insurance}}</li>    
        </ul>
        {{1 + 1}}
      </body>
    </html>
    ```
1. Following that same structure, show the appointments and patients on the index page

1. `touch Procfile` - `web: node server.js`
    - to deploy to heroku

1. run `nodemon server.js`
    - open localhost:4000






