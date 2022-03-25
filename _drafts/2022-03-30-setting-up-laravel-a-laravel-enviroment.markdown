---
layout: post
title:  "O jeito mais fácil de configurar um ambiente de desenvolvimento Laravel usando Docker no Ubuntu 20"
date:   2022-03-30 11:34:51 -0300
categories: laravel environment dev docker
---

## Introdução
Nesse artigo iremos configurar um ambiente de desenvolvimento utilizando o framework PHP [Laravel 9.x](https://laravel.com/docs/) com auxilo da ferramenta [Laravel Sail](https://laravel.com/docs/9.x/sail) para criaçao dos nossos [Docker containers](https://www.docker.com/resources/what-container/). Esse artigo tem um carater mais prático e por isso focamos em como construir o ambiente sem explicar cada conceito e/ou ferramenta. Mas não se preocupe caso você não esteja familiarizado com algum desses conceitos técnicos a maioria deles possuem links pra te ajudar a entender melhor sobre o que estamos falando. Outro ponto importante é que meu sistema operacional é o [Ubuntu Linux 20.4](https://releases.ubuntu.com/20.04/) então podem ter pequenas diferenças entre como executar os comandos, mas, se você seguir os passos indicados para o seu sistema operacional nos links de documentação não deve ter problemas pra seguir esse tutorial.

## Pré-requisitos
1. [GIT](https://git-scm.com/book/pt-br/v2/Come%C3%A7ando-Instalando-o-Git)
2. [CURL](https://curl.se/docs/install.html) 
3. [Docker](https://docs.docker.com/engine/install/#server)
4. [Docker Compose](https://docs.docker.com/compose/install/)

## Instalação do LaravelÍ
