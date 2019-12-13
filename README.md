Task1: Rating predictionRead prediction

Predict given a (user,book) pair from ‘pairs Read.txt’ whether the user would read the book (0 or 1).

Conclusion: My best prediction accuracy on the validation data is 69.9% and 70% on test using Logistic Regression.
2 features: mean Jaccard similarity of books and read proportion—one measures the similarity of the users and the other measures the popularity of the book. 

My reading prediction is top35% among 847 people in Kaggle.


Approach: 

1.	Prepare data 

(1) split the training data (‘train Interactions.csv.gz’) as follows: 

a. Reviews 1-190,000 for training 

b. Reviews 190,001-200,000 for validation

(2) For each entry (user,book) in the validation set, sample a negative entry by randomly choosing a book that user hasn’t read. 

The way to do this is to first list all the books. And for each user, list his/her read books. Next, use the all-book list minus the read-book list, which gives me the unread-list. Then, randomly choose (from library random import choice) a book from the unread-list.
So, the validation data becomes 10000( 1 ) +10000( 0 )=20000 books.
We need to balance data to avoid naive prediction.

2.	Build Model 

（a）Model A (baseline model): 

Find the most popular books that account for 50% of interactions in the training data(totalRead/2). Return ‘1’ whenever such a book is seen at test time, ‘0’ otherwise.
If the book is in the popular book list, predict 1.
Otherwise, predict 0.
The accuracy of the baseline model on validation set is 0.6431.
The reason is that the baseline model predicted much lesser 1 than 0. That means it’s too strict to define a book as “popular”. We need to loosen the criteria of judging a book as popular or not, so we can predict more 1 and increase the accuracy on the validation set.
I adjusted the threshold from 1.25~2 with the step of 0.05 to see the best threshold.
As a result, threshold of 1.6 is the best on validation data. That is to take about 67% of the popular books to predict 1.
The accuracy is increased to 0.6519

 
The problem of the model is that it doesn’t relate the user with the book. But in fact, the book and the user must have some interaction. So, we go further to use Jaccard Similarity, which computes the interactions of users and books. 

(b) Model B (Jaccard Similarity):

Given a pair (u, b) in the validation set, consider all training items b 0 that user u has read. For each, compute the Jaccard similarity between b and b 0 , i.e., users (in the training set) who have read b and users who have read b 0 . Predict as ‘read’ if the maximum of these Jaccard similarities exceeds a threshold.

picked a book that is read, the max Jaccard is 0.026. I used this as one reference to set the benchmark of Jaccard.
Loop a threshold from 0.001~0.03, and find when the threshold is 0.011, the accuracy is the best. It's 0.62.


(c) Model C (Regression Model):

My best prediction accuracy on the validation data is 69.9% and 70% on test using Logistic Regression. I tried 7 features and finally picked 2: mean Jaccard similarity of books and read proportion—one measures the similarity of the users and the other measures the popularity of the book.  

The other 5 features I’ve also tried but don’t work better are: 

Maximum Jaccard similarity of books 

Mean and max Jaccard similarity of users 

Mean Cosine similarity of books

Log of the two features 

Given a pair (u, b) in the validation set, consider all training items b’ that user u has read. For each, compute the Jaccard similarity between b and b’. And get mean Jaccard similarity as the feature.

Because the mean Jaccard similarity is about 0.0001~0.009, I try to balance the other feature—read proportion of the book on the same scale. So, I use the book’s total read time /(0.3*len(traindata)) as the popularity feature. 

The parameter 0.3 is also through trials.

When training the regression model, I also tried put higher weights to labels that equal to 1 because I found my prediction has much fewer 1s than 0s in a balanced data set. I think my model should predict more 1s. But, even I changed the class_weight={0:0.46, 1:0.54}, the prediction result on test data is not improved.

Task2: Rating prediction

I used the average rating +bias model: α + βuser + βitem to predict the rating of a book.

My rating prediction is top25% among 423 people in Kaggle.

First to initialize α, βuser and βitem, then, using the iteration convergence to calculate the bias terms for each user and each book until the MSE changes little. 

When fixing the lambda=1, I iterated 20 times until MSE began increasing a little. After that, I looped lambda from 1 to 7 to find the one that could minimize MSE on validation data. 

Finally, I chose lambda=3, the α and bias terms after 2nd iteration minimized the MSE.

So, I applied the α + βuser + βitem on the test data and predict the rating. 

One thing that I tried to improve is to pick different lambda when calculating βuser and βitem.

But it seems too heavy for my computer to run the for loop. 

Further improvement can be done with latent model.
