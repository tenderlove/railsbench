# railsbench

Based on [headius/pgrailsbench](https://github.com/headius/pgrailsbench),
but on Rails 5.2 and with database seeds.

## What's this?

This application is generated by following commands:

```
rails new railsbench --database=postgresql
rails g scaffold post title:string body:text published:boolean
```

## Setup

Make sure config/database.yml works for your local PostgreSQL.
Replace "k0kubun" with your $USER in the following instructions.

```bash
sudo apt install postgresql

sudo -u postgres psql
postgres=$ create role k0kubun superuser;

# Add `local all k0kubun trust`
sudo vi /etc/postgresql/10/main/pg_hba.conf

sudo systemctl restart postgresql
```

Populate database and assets for `RAILS_ENV=production`.

```bash
gem install bundler -v 1.17.2
bundle install
bundle exec rake db:create db:migrate db:seed RAILS_ENV=production
bundle exec rake assets:precompile RAILS_ENV=production
```

## Run

```bash
# MRI
bundle exec puma -e production --workers=4 --threads=1 --preload

# JRuby, TruffleRuby
bundle exec puma -e production --threads=4
```

You can see posts on `localhost:3000/posts`.

Assets are not shown properly with `-e production` but it's intentional because
Rails assumes we delegate assets distribution to a reverse proxy on production by default.

If you want to see the actual apprecation behavior, do the setup with `RAILS_ENV=development`
and try `puma -e development`.

## Benchmark

Install Apache Bench with `sudo apt install apache2-utils`. Then:

```bash
# HTML posts#index
ab -c 4 -n 10000 localhost:3000/posts

# JSON posts#index
ab -c 4 -n 10000 localhost:3000/posts.json

# HTML posts#show
ab -c 4 -n 10000 localhost:3000/posts/1

# JSON posts#show
ab -c 4 -n 10000 localhost:3000/posts/1.json
```

## Benchmark Results

Here is the benchmark result on: Intel 4.0GHz i7-4790K, 16GB memory, x86-64 Ubuntu 8 Cores

For each result, I measured results with following steps:

* Restart puma
* Run `ab -n 10000` as warmup
* Run `ab -n 1000` as benchmark

Following tables are showing response time milliseconds for each %ile in the last 1,000 requests.
See [log/benchmark](./log/benchmark) for details.

### GET /posts

|      | ruby 2.6.2 | ruby 2.6.6 JIT | JRuby 9.2.6.0 | JRuby 9.2.6.0 indy | JRuby 9.2.6.0 Graal |
|:-----|:-----------|:---------------|:--------------|:-------------------|:--------------------|
| 50%  | 11 | 23 | 18 | 13 | 22 |
| 66%  | 15 | 27 | 21 | 18 | 22 |
| 75%  | 16 | 30 | 26 | 19 | 22 |
| 80%  | 17 | 31 | 26 | 19 | 24 |
| 90%  | 18 | 41 | 28 | 20 | 28 |
| 95%  | 24 | 51 | 29 | 22 | 35 |
| 98%  | 28 | 61 | 32 | 26 | 36 |
| 99%  | 34 | 70 | 34 | 28 | 38 |
|100%  | 44 |101 | 41 | 35 | 44 |

### GET /posts.json

|      | ruby 2.6.2 | ruby 2.6.6 JIT | JRuby 9.2.6.0 | JRuby 9.2.6.0 indy | JRuby 9.2.6.0 Graal |
|:-----|:-----------|:---------------|:--------------|:-------------------|:--------------------|
| 50%  | 27 | 61 | 31 | 21 | 53 |
| 66%  | 29 | 69 | 32 | 23 | 58 |
| 75%  | 30 | 76 | 34 | 26 | 61 |
| 80%  | 31 | 81 | 35 | 28 | 62 |
| 90%  | 40 |106 | 40 | 31 | 68 |
| 95%  | 54 |117 | 45 | 32 | 78 |
| 98%  | 56 |131 | 47 | 35 | 85 |
| 99%  | 62 |144 | 50 | 38 | 92 |
|100%  | 78 |203 | 56 | 45 |101 |

### GET /posts/1

|      | ruby 2.6.2 | ruby 2.6.6 JIT | JRuby 9.2.6.0 | JRuby 9.2.6.0 indy | JRuby 9.2.6.0 Graal |
|:-----|:-----------|:---------------|:--------------|:-------------------|:--------------------|
| 50%  | 3 |  5 |  8 |  7 | 8 |
| 66%  | 4 |  6 |  9 |  8 | 9 |
| 75%  | 4 |  7 |  9 |  9 |12 |
| 80%  | 5 |  8 | 10 | 10 |13 |
| 90%  | 5 | 10 | 13 | 12 |16 |
| 95%  | 6 | 12 | 15 | 14 |19 |
| 98%  | 9 | 17 | 19 | 18 |22 |
| 99%  |14 | 22 | 22 | 19 |26 |
|100%  |28 | 31 | 28 | 46 |28 |

### GET /posts/1.json

|      | ruby 2.6.2 | ruby 2.6.6 JIT | JRuby 9.2.6.0 | JRuby 9.2.6.0 indy | JRuby 9.2.6.0 Graal |
|:-----|:-----------|:---------------|:--------------|:-------------------|:--------------------|
| 50%  | 3 |  4 |  8 |  7 |  6 |
| 66%  | 3 |  6 |  9 |  7 |  7 |
| 75%  | 4 |  7 |  9 |  8 |  8 |
| 80%  | 4 |  7 | 10 |  9 |  8 |
| 90%  | 5 |  9 | 13 | 11 |  9 |
| 95%  | 5 | 13 | 16 | 11 | 10 |
| 98%  | 6 | 17 | 19 | 13 | 10 |
| 99%  |11 | 21 | 19 | 16 | 12 |
|100%  |29 | 29 | 28 | 20 | 15 |

## License

MIT License
