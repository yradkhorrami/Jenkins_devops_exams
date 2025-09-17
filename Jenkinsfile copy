pipeline
{
	agent any
	options { timestamps() }

	environment
	{
		DOCKER_ID	= 'yradkhorrami'
		TAG			= "v.${BUILD_ID}.0"
		MOVIE_IMG	= "${DOCKER_ID}/movie-service"
		CAST_IMG	= "${DOCKER_ID}/cast-service"
	}

	stages
	{
		stage('Checkout')
		{
			steps { checkout scm }
		}

		stage('Docker Build: movie-service')
		{
			steps
			{
				sh '''
				#!/bin/bash
				set -eux
				docker build -t "$MOVIE_IMG:$TAG" ./movie-service
				'''
			}
		}

		stage('Docker Build: cast-service')
		{
			steps
			{
				sh '''
				#!/bin/bash
				set -eux
				docker build -t "$CAST_IMG:$TAG" ./cast-service
				'''
			}
		}

		stage('Docker Push (both)')
		{
			environment { DOCKER_PASS = credentials('DOCKER_HUB_PASS') }
			steps
			{
				sh '''
				#!/bin/bash
				set -eux
				echo "$DOCKER_PASS" | docker login -u "$DOCKER_ID" --password-stdin
				docker push "$MOVIE_IMG:$TAG"
				docker push "$CAST_IMG:$TAG"
				'''
			}
		}
	}
}
