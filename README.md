# README

<a href="https://www.udacity.com/">
  <img src="https://s3-us-west-1.amazonaws.com/udacity-content/rebrand/svg/logo.min.svg" width="300" alt="Udacity logo svg">
</a>

Udacity Full Stack Web Developer Nanodegree program

[Project 6. Linux server deployment](https://github.com/br3ndonland/udacity-fsnd-p6-server)

Brendon Smith

[br3ndonland](https://github.com/br3ndonland)

[![license](https://img.shields.io/badge/license-MIT-blue.svg?longCache=true&style=for-the-badge)](https://choosealicense.com/)

## Description

In this project, I configured an Ubuntu Linux server instance on [DigitalOcean](https://www.digitalocean.com/) and deployed a Flask app to the server.

## Details

- DigitalOcean server instance: **udacity6**
- URL:
- Public IP address: 192.241.141.20
- SSH port: 2200
- User: `grader`
- Password: `grader`
- SSH key: Provided to Udacity reviewer during project submission
- Log in with

  ```shell
  ssh grader@192.241.141.20 -p 2200
  ```

- Software:
  - [Flask catalog application](https://github.com/br3ndonland/udacity-fsnd-p4-flask-catalog)
  - Python 3
  - pip
- See [server-methods.md](info/server-methods.md) for walkthrough.