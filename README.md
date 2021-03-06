# Send Me Files

This is the tiniest, cheapest way to let people send you huge files
for cents per month.

__Send Me Files__ lets people send you large files from their
browser, without the file size constraints of email and without talking them
into installing Dropbox or Google Drive or whatever.

It's a Rails 4 app, so you can easily deploy it (for free!) to Heroku or a similar provider.

# Screenshots

<img src="http://f.cl.ly/items/420w0w0c403E2C3E090W/Screen%20Shot%202014-03-05%20at%2022.35.21.png">

__It works from iOS 6 and 7, too:__

<img src="http://f.cl.ly/items/0W420j0T203B1q2w143N/Pulsar%20Mar%205,%202014,%2022%3A37%3A13.png">

When people upload a file, you get an email:

<img src="http://f.cl.ly/items/3p2J3H2n0s3F47050h3T/Screen%20Shot%202014-03-05%20at%2020.57.31.png">

# Deploying Your Own

1. Clone (perhaps even fork and clone) this repository.
2. Create a Heroku account if you don't have one, or get an account with a similar provider.
3. Create an AWS account if you don't already have one.
4. Use the AWS Console to create an S3 bucket (for storing the uploaded files.)
5. Create an SNS topic (for notifying you of new files.)
6. Create an IAM user for your application to use. Give it this policy:

        {
          "Version": "2012-10-17",
          "Statement": [
            {
              "Sid": "Stmt1393987487000",
              "Effect": "Allow",
              "Action": [
                "s3:*"
              ],
              "Resource": [
                "arn:aws:s3:::your-bucket",
                "arn:aws:s3:::your-bucket/*"
              ]
            },
            {
              "Sid": "AllowSendingSNS",
              "Effect": "Allow",
              "Action": [
                "sns:publish"
              ],
              "Resource": [
                "arn:aws:sns:your-sns-topic"
              ]
            }
          ]
        }

7. Add your own email address as a subscriber to the SNS topic. (Or subscribe via text messages, which is even more awesome.)
8. Generate a session secret with `rake secret`. You'll provide this to the application as an environment variable called `SECRET_KEY`.
8. Deploy this application to Heroku or whatever (perhaps `git push heroku master`).
9. Define the required enviroment variables using `heroku config:set` or whatever your provider calls it.

    Supply the app with the key and secret of the IAM user by defining
    `AWS_KEY` and `AWS_SECRET` as environment variables.
    
    You also need to specify a `HOSTNAME` so the application can create
    the proper CORS policy for the S3 bucket.
    
    Specify the full ARN of your SNS topic as an environment variable
    called `SNS_TOPIC`.
    
    Provide the app with the session key you produced with `rake secret`
    by defining `SECRET_KEY`.
    
    When deployed, you need to supply all of these environment variables:
    
        AWS_KEY=your-key
        AWS_SECRET=your-secret
        HOSTNAME=yourdomain.com
        S3_BUCKET=your-bucket
        SNS_TOPIC=arn:aws:sns:us-east-1:1234567:your-sns-topic
        SECRET_KEY=abcdef123456

    Optionally, you can specify an access password that must be supplied as
    a query parameter called `a` in order to upload files:

        PASSWORD=someSecretPassword
    
    For this example, people would need to go to `your-app.herokuapp.com/?a=someSecretPassword` to be able to upload files. __This is designed to keep out people who stumble on the page, not to provide security.__
    
    Also, you can define an environment variable called NAME to make the app display your name rather than just "me".
    
        NAME=Jeffrey

10. Once your app is deployed and the configuration variables are defined, configure the bucket to accept cross-origin requests by running `rake deployment:configure_bucket` from your production environment (probably with `heroku run`.)

