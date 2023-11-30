# Rapport

Mini-projet carnet de voyage :

Projet réalisé en Ruby. 

La base de données est une SQLite.

## Etape 1 : Configuration de base 

* Création d'un nouveau projet : 
```ruby
 rails new CarnetDeVoyage
```
* Création du modèle user
```ruby
rails g model User name:string email:string
```
* Pour créer le modèle trip, comme il nécessite une clé étrangère je l'ajoute directement dans la définition du modèle
```ruby
rails g model Trip destination:string description:string start_date:string end_date:string user:belongs_to
```
* Je génère ensuite le controller pour user afin de manipuler les données de la table user avec index à la fin pour créer la fonction index
```ruby
 rails generate controller Users index
```
* Je génère ensuite le controller trip pour la même raison que celui de user
```ruby
rails generate controller Trips index
```
* Je run ensuite les migrations en allant directement sur le navigateur et en cliquand sur run migrations
### Ajout de data dans la base de données
* Ouverture de la console rails
```ruby
rails c
```
* Ajout de data dans User
```ruby
User.create name: "Jean", email: "Jean@mail.com"
```
* Ajout de data d'un trip avec comme clé étrangère le premier user
```ruby
Trip.create description: "Description du trip", destination: "Tokyo", start_date: "15 novembre 2023", end_date: "30 novembre 2023", user: User.first
```
* Pour créer une bonne relation entre les tables trip et user il faut ajouter dans le model User
```ruby
class User < ApplicationRecord
    has_many :trips
end
```
## Etape 2 Manipulation de données
* créer une route pour user, dans routes.rb
```ruby
get '/users', to: 'users#index'
```
* même chose pour trip, dans routes.rb
```ruby
get '/trips', to: 'trips#index'
```

### CRUD trips

* Création de la première vue pour afficher tous les trips (localhost:3000/trips)
```erb
<table>
    <thead>
        <tr>
            <th>Destination</th>
            <th>Description</th>
            <th>Début</th>
            <th>Fin</th>
            <th>User associé</th>
        </tr>
    </thead>
    <tbody>
        <% @trips.each do |trip| %>
        <tr>
            <th>
                <%= trip.destination %>
            </th>
            <th>
                <%= trip.description %>
            </th>
            <th>
                <%= trip.start_date %>
            </th>
            <th>
                <%= trip.end_date %>
            </th>
            <th>
                <%= trip.user.id %>
            </th>
        </tr>
    </tbody>
    <% end %>
</table>

```
* Controller pour récupérer tout les trips : 
```ruby
  def index
    @trips = Trip.all
  end
```
* Création de la vue pour visualiser un trip, il faut créer dans les view trips la view show.html.erb qui sera lié à la fonction show dans le controller trip
```erb
<div>
    Voici le voyage correspondant à cet id
    <table>
        <thead>
            <tr>
                <th>Destination</th>
                <th>Description</th>
                <th>Début</th>
                <th>Fin</th>
                <th>User associé</th>
            </tr>
        </thead>
        <tbody>
            <tr>
                <th>
                    <%= @trip.destination %>
                </th>
                <th>
                    <%= @trip.description %>
                </th>
                <th>
                    <%= @trip.start_date %>
                </th>
                <th>
                    <%= @trip.end_date %>
                </th>
                <th>
                    <%= @trip.user.id %>
                </th>
            </tr>
        </tbody>
    </table>
</div>
```
* Création de la fonction show dans le controller trip
```ruby
  def show
    @trip = Trip.find(params[:id])
  end
```
* création de la vue pour modifier un trip edit.html.erb, j'utilise from_with pour générer un formulaire en html, ensuite chaque input est généré avec form.{type_input} ainsi que le nom du champ qu'il permet de modifier. Avec ceci les champs sont prérempli automatiquement. J'utilise également form.collection qui reprend les user pour faire en sorte d'avoir une liste  déroullante qui reprend tous les user et permet de lié le trip au user facilement
```erb
<!-- app/views/trips/edit.html.erb -->
<h1>Modifier le voyage</h1>

<%= form_with(model: @trip, local: true) do |form| %>
    <%= form.hidden_field :id %>
  <div class="field">
    <%= form.label :destination %>
    <%= form.text_field :destination %>
  </div>

  <div class="field">
    <%= form.label :description %>
    <%= form.text_area :description %>
  </div>

  <div class="field">
    <%= form.label :start_date %>
    <%= form.text_field :start_date %>
  </div>

  <div class="field">
    <%= form.label :end_date %>
    <%= form.text_field :end_date %>
  </div>

  <div class="field">
    <%= form.label :user_id %>
    <%= form.collection_select :user_id, User.all, :id, :name %>
  </div>

  <div class="actions">
    <%= form.submit "Enregistrer les modifications" %>
  </div>
<% end %>

<%= link_to 'Retour', trips_path %> <!-- permet de générer un <a> qui ramène vers la liste des trips -->
```
* Dans le controller il faut ajouter la fonction pour modifier le trip en question, en 2 temps la première est edit qui receptionne la demande et update qui effectue la mise à jour
```ruby
  def edit
  end

  def update
    @trip = Trip.find(params[:id]) #permet de retrouver le trip à modifier
    if @trip.update(trip_params)
      redirect_to @trip, notice: 'Le voyage a été mis à jour avec succès.'
    else
      render :edit
    end
  end
```
* Afin de simplifier les choses dans les routes j'ai changé toutes les routes des trips par une simple ligne qui permet de générer automatiquement toutes les routes nécessaire.
```ruby
  resources :trips
```
* Afin de simplifier les choses dans le controller et ne pas avoir à rechercher le trip quand j'en ai besoins dans les fonctions, j'ai créer une fonction qui permet de le faire 
```ruby
  private

  def set_trip
    @trip = Trip.find(params[:id])
  end
```
* Je l'utilise automatiquement dans le controller sur les fonction qui le nécessite, ainsi je n'ai plus à rechercher le trip dans chaque fonction, ce sera fait automatiquement
```ruby
  before_action :set_trip, only: [:show, :edit, :update, :destroy]
```
* Dans la view qui liste tous les trips j'ai ajouté dans la table le lien pour modifier le trip 
```erb
<th>
    <%= link_to 'Modifier', edit_trip_path(trip) %>
</th>
```
* Même chose pour la view qui donne un trip
```erb
<th>
    <%= link_to 'Modifier', edit_trip_path(@trip) %>
</th>
```
* Maitenant je vais ajouter la suppression, première chose c'est de créer dans le controller la fonction qui va permettre de delete, comme j'ai déjà mis la fonction set_trip qui va récupérer le trip j'ai juste à mettre 
```ruby
  def destroy
    @trip.destroy
    redirect_to trips_url, notice: 'Le voyage a été supprimé avec succès.'
  end
```
* la route étant également déjà faite avec le resources :trips, je n'ai qu'à ajouter un bouton pour delete le trip dans ma view de tous les trips
```erb
<th>
    <%= button_to 'Delete', trip_path(trip), :method => :delete %>
</th>
```
* même chose pour la view qui permet d'afficher un seul trip
```erb
<th>
    <%= button_to 'Delete', trip_path(@trip), :method => :delete %>
</th>
```
* Pour créer un voyage il créer une view new.html.erb avec ceci dedans, le formulaire reprend le même principe que celui de update
```erb
<!-- app/views/trips/_form.html.erb -->
<%= form_with(model: @trip, local: true) do |form| %>
  <% if @trip.errors.any? %>
    <div id="error_explanation">
      <h2><%= pluralize(@trip.errors.count, "erreur") %> ont empêché la sauvegarde de ce voyage :</h2>

      <ul>
        <% @trip.errors.full_messages.each do |message| %>
          <li><%= message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= form.label :destination %>
    <%= form.text_field :destination %>
  </div>

  <div class="field">
    <%= form.label :description %>
    <%= form.text_area :description %>
  </div>

  <div class="field">
    <%= form.label :start_date %>
    <%= form.text_field :start_date %>
  </div>

  <div class="field">
    <%= form.label :end_date %>
    <%= form.text_field :end_date %>
  </div>

  <div class="field">
    <%= form.label :user_id %>
    <%= form.collection_select :user_id, User.all, :id, :name %>
  </div>

  <div class="actions">
    <%= form.submit "Créer le voyage" %>
  </div>
<% end %>
```
* Ensuite il reste à mettre la bonne fonction dans le controller pour créer le trip en question
```ruby
  def new
    @trip = Trip.new
  end

  def create
    @trip = Trip.new(trip_params)

    if @trip.save
      redirect_to @trip, notice: 'Le voyage a été créé avec succès.'
    else
      render :new
    end
  end
```
* Il ne reste plus qu'a aller sur /trips/new pour voir le form et la view et enregistrer un nouveau trip

