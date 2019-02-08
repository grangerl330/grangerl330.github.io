---
layout: post
title:      "My first Ruby cli gem"
date:       2019-02-07 15:22:02 -0500
permalink:  my_first_ruby_cli_gem
---


Today I completed my first Ruby CLI gem for my final project in the Object Oriented Ruby section the online software engineering course. It was a fun, challenging project that required use of most of the knowledge I have learned so far, as well as new knowledge learned along the way. I really enjoyed the process of using the skills I have learned to build an actual functioning program. Overall I felt like completing this project was a great learning experience and a step forward in my journey to become a programmer.

The gem I built is called ATP Rankings Top 100. It is a tool that allows the user to see the current top 100 rankings list from the www.atptour.com , the official site of Men's professional tennis, and stats/info on each player in that rankings list. The tool is CLI or command line interface, which means that it is run in the terminal through user's inputs.

At the start of this build, the first thing I did was determine the structure of the files and how they would need to interact with each other. I used the ruby Bundler to create a basic layout of the files in the program, then added the additional files I would need. There were player.rb, scraper.rb, and cli.rb. Each file was used to house one of the 3 classes I needed, player, scraper, and cli. Below I'll go through each of these classes and how they make the program work.


### Lets start with the Scraper class:
```
class AtpRankingsTop100Cli::Scraper

  def scrape_players
    @doc = Nokogiri::HTML(open("https://www.atptour.com/en/rankings/singles"))

    @doc.search("tbody tr").each do |player_tr|
      player = AtpRankingsTop100Cli::Player.new
      player.name = player_tr.search(".player-cell").text.strip
      player.points = player_tr.search(".points-cell").text.strip
      player.age = player_tr.search(".age-cell").text.strip
      player.rank = player_tr.search(".rank-cell").text.strip
      player.country = player_tr.search(".country-cell img").attr('alt')
      player.num_tourns_played = player_tr.search(".tourn-cell").text.strip
      player.bio_link = "https://www.atptour.com#{player_tr.search(".player-cell a").attr('href')}"
    end
  end

end

```

In order to retrieve the information about the rankings list and players, the program needs to scrape the data from https://www.atptour.com/en/rankings/singles and save that data into something. I used a ruby gem called Nokogiri to accomplish this. Nokogiri essentially retrieves all of the HTML data from a website and allows you to select certain pieces of info from that data using css selectors. Using Nokogiri, the Scraper class selects the relevant data from the website and saves each piece of data to an attribute of a newly created instance of the Player class. This allows the program to call for specific attributes of each player in other sections of the program.

### Next lets look at the Player class: 

```
class AtpRankingsTop100Cli::Player
  attr_accessor :name, :rank, :age, :points, :country, :num_tourns_played, :bio_link, :turned_pro, :weight, :height, :birthplace, :residence, :plays, :coach

  @@all = []

  def initialize
    @@all << self
  end

  def self.all
    @@all
  end

  def self.list(start_of_range, end_of_range)
    puts "Ranks #{start_of_range + 1} - #{end_of_range + 1}:"
    @@all[start_of_range..end_of_range].each_with_index {|player, index| puts "#{index + start_of_range + 1}. #{player.name}"}
  end

  def self.find(id)
    @@all[id-1]
  end

  def doc
    @doc ||= Nokogiri::HTML(open(self.bio_link))
  end

  def turned_pro
    @turned_pro = doc.css(".table-big-value")[1].text.strip
  end

  def weight
    @weight ||= doc.css(".table-big-value")[2].text.strip
  end

  def height
    @height ||= doc.css(".table-big-value")[3].text.strip
  end

  def birthplace
    @birthplace ||= doc.css(".table-value")[0].text.strip
  end

  def residence
    @residence ||= doc.css(".table-value")[1].text.strip
  end

  def plays
    @plays ||= doc.css(".table-value")[2].text.strip
  end

  def coach
    @coach ||= doc.css(".table-value")[3].text.strip
  end

end

```

The Player class is set up with an attribute that corresponds to each piece of data that the program will need to show the user. Several of those attributes are set by the Scraper class, while some are set in in the Player class itself. This is done because the attributes that are set in the Player class are obtained from a player secific page rather than the main rankings page https://www.atptour.com/en/rankings/singles. If I set up the Scraper class to set these attributes, the load time of the program would be much longer, because it would have to look through 100 different player pages for the relevant data all at once. Setting it up this way allows for the program to scrape the data from a player specific page only when the user asks to see that specific player, which greatly decreases the initial load time. The use of `||=` in each attribute methods allows the program to only scrape that player's specific page data one time. So if the user looks a player, then goes back to look at that player later, the program will not need to scrape that data again. 

### Next lets look at the CLI class:

```
class AtpRankingsTop100Cli::CLI

  def call
    puts
    puts "Welcome to ATP Rankings Top 100!"
    puts
    puts "Type exit at any time to exit the program"
    AtpRankingsTop100Cli::Scraper.new.scrape_players
    start
  end

  def start
    range_choice
    list_range
  end

  def range_choice
    puts
    puts "Rankings Ranges:"
    puts "1-10"
    puts "11-20"
    puts "21-30"
    puts "31-40"
    puts "41-50"
    puts "51-60"
    puts "61-70"
    puts "71-80"
    puts "81-90"
    puts "91-100"
    puts
    puts "Type a number in the range you would like to see or type all to see all 100 players"
    @range_choice_input = gets.chomp
  end

  def list_range
    if (1..100).include?(@range_choice_input.to_i)
      puts
      case @range_choice_input.to_i
      when (1..10)
        AtpRankingsTop100Cli::Player.list(0, 9)
        @range_checker = (1..10).to_a
      when (11..20)
        AtpRankingsTop100Cli::Player.list(10, 19)
        @range_checker = (11..20).to_a
      when (21..30)
        AtpRankingsTop100Cli::Player.list(20, 29)
        @range_checker = (21..30).to_a
      when (31..40)
        AtpRankingsTop100Cli::Player.list(30, 39)
        @range_checker = (31..40).to_a
      when (41..50)
        AtpRankingsTop100Cli::Player.list(40, 49)
        @range_checker = (41..50).to_a
      when (51..60)
        AtpRankingsTop100Cli::Player.list(50, 59)
        @range_checker = (51..60).to_a
      when (61..70)
        AtpRankingsTop100Cli::Player.list(60, 69)
        @range_checker = (61..70).to_a
      when (71..80)
        AtpRankingsTop100Cli::Player.list(70, 79)
        @range_checker = (71..80).to_a
      when (81..90)
        AtpRankingsTop100Cli::Player.list(80, 89)
        @range_checker = (81..90).to_a
      when (91..100)
        AtpRankingsTop100Cli::Player.list(90, 99)
        @range_checker = (91..100).to_a
      end
      menu
    elsif @range_choice_input.downcase == "all"
      puts
      AtpRankingsTop100Cli::Player.list(0, 99)
      @range_checker = (1..100).to_a
      menu
    elsif @range_choice_input.downcase == "exit"
      goodbye
    else
      puts
      puts "Invalid Input"
      start
    end
  end

  def menu
    puts
    puts "Enter player number for more information:"
    @menu_input = gets.chomp
    @player = AtpRankingsTop100Cli::Player.find(@menu_input.to_i)

    if @range_checker.include?(@menu_input.to_i)
      more_info
      see_additional_info(@player)
      stay_in_range
    elsif @menu_input.downcase == "exit"
      goodbye
    else
      puts
      puts "Invalid Input"
      list_range
    end
  end

  def stay_in_range
    puts
    puts "Would you like to stay in this rankings range? [y/n]"
    @stay_in_range_input = gets.chomp

    if @stay_in_range_input.downcase == "y"
      list_range
    elsif @stay_in_range_input.downcase == "n"
      start
    elsif @stay_in_range_input.downcase == "exit"
      goodbye
    else
      puts
      puts "Invalid Input"
      stay_in_range
    end
  end

  def more_info
    puts
    puts "Name: #{@player.name}"
    puts "Age: #{@player.age}"
    puts "Country: #{@player.country}"
    puts "Rank: #{@player.rank}"
    puts "Points: #{@player.points}"
    puts "Tournaments Played: #{@player.num_tourns_played}"
  end

  def see_additional_info(player)
    puts
    puts "Would you like to see additional info on this player? [y/n]"
    @see_additional_info_input = gets.chomp

    if @see_additional_info_input.downcase == "y"
      puts
      puts "Additional Info:"
      puts
      puts "Turned pro: #{@player.turned_pro}"
      puts "Weight: #{@player.weight}"
      puts "Height: #{@player.height}"
      puts "Birthplace: #{@player.birthplace}"
      puts "Residence: #{@player.residence}"
      puts "Plays: #{@player.plays}"
      puts "Coach: #{@player.coach}"
    elsif @see_additional_info_input.downcase == "n"
      stay_in_range
    elsif @see_additional_info_input.downcase == "exit"
      goodbye
    else
      puts
      puts "Invalid Input"
      see_additional_info(@player)
    end
  end

  def goodbye
    puts
    puts "Thank you for using ATP Rankings Top 100!"
    puts
    exit
  end

end

```

The CLI class is all about actually running the functions of the program. When the user runs the program, it starts by calling the `call` method on the CLI class. This method starts off the chain of logic within the CLI class that makes the program function.  The basic structure of the program is as follows:

1. Initial load sequence

2. Ask the user which ranking range they want to see

3. List the chosen range

4. Ask the user which player they would like to see within the chose range

5. Ask the user if they would like to see more info on that chose player

6. Ask the player if they would like to stay in the same range they are currently in

7. Repeat

Each step of this structure is performed by a different method in the CLI class. Each method contains conditionals that peform different functions based on the user's input. Those functions include running a method to advance further in the program structure, running a method to go backward in the structure, telling the user their input was invalid, or exiting the program. Each method is able to account for any type of input the user enters, so if there is a typo or the wrong number is entered, the program will give an appropriate response. 


And thats it, those 3 classes make the entire program function! While this is program is quite simple compared to whats possible with Ruby, it was a great start for a beginner like me and a great lesson on how to get different Ruby classes to work together to accomplish a goal. Hopefully in the future I will be able to use this base level of knowledge to build something bigger and better, but for now I can officially say that I have created my own Ruby gem!

























 

