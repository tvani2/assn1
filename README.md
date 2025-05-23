კონკურსის აღწერა:
    კონკურსის მიზანია ისეთი მოდელის ტრენინგი, რომელიც მოცემული feature-ების გამოყენებით შეძლებს სახლების ფასების პროგნოზირებას.


ჩემი მიდგომა:
    შევისწავლე train.csv ფაილი და ეტაპობრივად გავაკეთე მონაცემების cleaning, feature engineering და feature selection. საბოლოოდ სხვადასხვა მოდელების ტრენინგის შემდეგ ავარჩიე საუკეთესო, რომელმაც 0.13137 RMSLE შედეგი აჩვენა.


რეპოზიტორიის სტრუქტურა:
    model_experiment.ipynb:
ფაილი მოიცავს სრულ ანალიტიკურ პროცესს. თითოეული ეტაპი, როგორც ტექნიკური, ისე ლოგიკური თვალსაზრისით, სათაურებითაა გამოყოფილი და ახსნილია.
    model_inference.ipynb:
აქ ტესტ მონაცემებზე ხდება პროგნოზის აგება შერჩეული საუკეთესო მოდელის გამოყენებით. რადგან pipeline არ გამოვიყენე, ხელით მომიწია მონაცემების cleaning, feature engineering, feature selection, შემდეგ კი ტრენინგიც. შედეგად შეიქმნა submissions.csv, რომელიც ავტვირთე Kaggle-ზე.
    README.md:
შეიცავს სრულად აღწერილ მიდგომებს, გამოყენებულ მეთოდებსა და ძირითადი ნაბიჯების ახსნას.


Feature Engineering:
    train და test მონაცემები ცალკე დავყავი.
    სვეტები, რომელთაც 80%-ზე მეტი NaN ჰქონდა, ამოვიღე. დანარჩენი შევავსე სხვადასხვა წესით — ზოგჯერ განაწილების ან მსგავსი სვეტის მონაცემების მიხედვით. მაგალითად, LotFrontage სვეტის NaN მნიშვნელობები შევავსე LotArea-ის მიხედვით, რადგან მათ შორის კორელაცია შეინიშნებოდა. იგივე წესით შევავსე test-სეტის შესაბამისი ველები.
    სვეტები დავყავი კატეგორიულ (cat) და რიცხვით (num) ჯგუფებად. ზღვრად ავიღე კატეგორიის მაქსიმუმ 3 განსხვავებული მნიშვნელობა.
    გამოვთვალე WOE/IV მნიშვნელობები, რათა უკეთ გამერჩია ინფორმაციულად სასარგებლო სვეტები.


Feature Selection:
    გამოვიყენე კორელაციის ფილტრი — თუ ორი სვეტის ერთმანეთთან კორელაცია > 0.8 იყო, ერთ-ერთს ვშლიდი იმის მიხედვით, რომელს ჰქონდა უფრო ნაკლები კავშირი SalePrice-თან.
    გამოყენებული მაქვს RFE (Recursive Feature Elimination) სხვადასხვა feature_count-ით, როგორც R²-ის, ისე RMSE-ის მაქსიმიზაციის მიზნით. ოპტიმალური აღმოჩნდა 20 სვეტი.
    დავდროპე ისეთი სვეტები, რომელთა Value Distribution უკიდურესად გაუწონასწორებელი იყო. მაგ.: Street სვეტში Pave გვხვდებოდა 1164-ჯერ, ხოლო Grvl მხოლოდ 4-ჯერ — ასეთი დისბალანსი ტრენინგს აფერხებს.


Training:
    გამოვიყენე როგორც წრფივი, ასევე არაწრფივი მოდელები:
L1 (Lasso):
    იმის გამო, რომ ბევრი სვეტი იყო დარჩენილი და მინდოდა Overfitting-ის თავიდან არიდება, Lasso ვცადე სხვადასხვა სკეილერზე: Standard, MinMax, Robust და None. ალფას მნიშვნელობები იყო: [0.001, 0.01, 0.1, 1, 10], ხოლო როგორც cyclic ასევე random.
    5-Fold Cross Validation-ის მიხედვით საუკეთესო შედეგი ჰქონდა α = 10-ზე, რის შემდეგაც დავტესტე logspace(0, 3, 20) დიაპაზონი და საუკეთესო აღმოჩნდა α = 54. გრაფიკზე ჩანს α-სა და RMSE-ს შორის  დამოკიდებულება.
    
L2 (Ridge):
    იგივე პრინციპით დავტესტე Ridge. α მოვძებნე logspace(-3, 3, 50)-ში. შედეგები ოდნავ უარესი იყო L1-თან შედარებით, ამიტომ ElasticNet-ს მივუბრუნდი.
ElasticNet:
    საუკეთესო შედეგი მივიღე l1_ratio = 0.7-ზე(შერწყმული მიდგომა L1 და L2-ს შორის)  
Ridge მხოლოდ RFE-ს შერჩეულ სვეტებზე:
    იმის გათვალისწინებით, რომ L2 მოდელი არ შლის სვეტებს, მხოლოდ ამცირებს მათ წონებს, გადავწყვიტე მეცადა Ridge მხოლოდ იმ სვეტებზე, რომლებიც RFE-მ დატოვა. თუმცა შედეგი მაინც დაბალი იყო.
Decision Trees:
    სხვადასხვა სიღრმისა და split-ის ხეებზე გავტესტე, მაგრამ შედეგები არადამაკმაყოფილებელი აღმოჩნდა.
Random Forest:
    ბევრად უკეთესი შედეგები აჩვენა. გრაფიკზე ნათლად ჩანს R² ზრდა წერტილების მატებასთან ერთად.
Gradient Boosting Regressor:
    ყველაზე კარგი შედეგი მივიღე.
    შევცვალე loss ფუნქცია huber-ით — რათა შემეცირებინა outlier-ების გავლენა მოდელზე (Huber loss უფრო გამძლეა outlier-ების მიმართ).
    ასევე გავზარდე n_estimators და გამოვიყენე Early Stopping — რათა მენახა როდის წყვეტს მოდელი გაუმჯობესებას და თავიდან ამერიდებინა overfitting.


Hyperparameter ოპტიმიზაცია:
    გამოვიყენე Grid Search, მეტრიკებად ავიღე RMSE და R². პარალელურად ვიზუალურად ვაკვირდებოდი train და validation შედეგებს შორის სხვაობას. მიზანი იყო მოდელის გენერალიზაციის უნარის შენარჩუნება   და overfitting-ის თავიდან აცილება.

საბოლოო მოდელის შერჩევის დასაბუთება:
    თითოეული მოდელისთვის ვეძებდი ოპტიმალურ პარამეტრებს და ვაკვირდებოდი როგორ ართმევდა თავს overfitting-ს. საბოლოოდ Gradient Boosting-მა აჩვენა საუკეთესო ბალანსი სიზუსტესა და სტაბილურობას შორის. გრაფიკებზე ჩანს, რომ უმრავლეს შემთხვევაში ფასის პროგნოზი ძალიან ახლოს იყო რეალურ მონაცემებთან. ასევე, train და test შედეგებს შორის სხვაობა მხოლოდ ≈0.05 იყო, რაც მიუთითებს მოდელის მაღალი გენერალიზაციის უნარზე.


MLflow Tracking: 
    MLflow ექსპერიმენტების ბმული: 
https://dagshub.com/tvani2/assn1.mlflow/#/experiments/4?searchFilter=&orderByKey=attributes.start_time&orderByAsc=false&startTime=ALL&lifecycleFilter=Active&modelVersionFilter=All+Runs&datasetsFilter=W10%3D 
ჩაწერილი მეტრიკების აღწერა:
    RMSE - საშუალო აცდენა predicted და რეალურ ფასს შორის
    R2 - [0, 1] რეინჯში რამდენად კარგად აკეთებს პრედიქციას

საუკეთესო მოდელის შედეგები:
    საბოლოო შედეგი მივიღე 0.13137.  model_experiment.ipynb -ში ბოლო პლოტზე ჩანს residuals, რაც ახლოსაა 0-თან. 
