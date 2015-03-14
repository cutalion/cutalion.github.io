---
layout: post
title: "Make all files in S3 directory private or public"
date: 2013-10-03 01:24
comments: true
categories: 
---

Once I've got a task to make a lot of files stored on S3 private.
In my project I used [carrierwave](https://github.com/carrierwaveuploader/carrierwave) and [fog](https://github.com/fog/fog).  
All files were public by default.

In order not to make many requests to AWS, I've decided to change [bucket policy](http://docs.aws.amazon.com/AmazonS3/latest/dev/UsingBucketPolicies.html) instead of ACL of each file.


{% highlight ruby %}
require 'fog'

class Storage
  def make_directory_private!(dirname)
    policy    = s3.get_bucket_policy bucket
    statement = private_statement dirname

    return if policy['Statement'].include? statement

    policy['Statement'].push statement
    s3.put_bucket_policy bucket, policy
  end

  def make_directory_public!(dirname)
    policy    = s3.get_bucket_policy bucket
    statement = private_statement dirname

    return unless policy['Statement'].include? statement

    policy['Statement'].delete statement
    s3.put_bucket_policy bucket, policy
  end

  def private_statement(dirname)
    {
      "Sid"       => "Private#{dirname}",
      "Effect"    => "Deny",
      "Principal" => {"AWS" => "*"},
      "Action"    => "s3:GetObject",
      "Resource"  => "arn:aws:s3:::#{bucket}/#{dirname}/*"
    }
  end

  def bucket
    "*************"
  end

  def s3
    @s3 ||= Fog::Storage.new({
      provider: 'AWS',
      aws_access_key_id: '**************',
      aws_secret_access_key: '***************'
    })
  end
end

Storage.new.make_directory_private!('user_42')
Storage.new.make_directory_public!('user_42')
{% endhighlight %}
