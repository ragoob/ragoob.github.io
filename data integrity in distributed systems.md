
عن ال data integrity فى الأنظمة الموزعة وال stream processing وأشياء أخرى 
لما يكون عندنا داتا ستور واحدة فى السيستم غالبا مشاكل ال concurrency مبتبئاش ضيف عندنا لكن تخيل لو شغالين على نظام موزع اكتر من داتا ستور معانا محتاجين ن replicate فيهم البيانات وطبعا ممكن يكونوا مختلفين
النوع SQL , index , cache فى ال distributed system ال ACID والحاجات الجميلة دى مش معانا لان عندنا اكتر من behavior خارج عن الابلكشن بتاعنا زى 
Network , Node , Clock (time behavior)
كل دى عوائق هتقابلنا هتتحكم فى ال trade offs مابين CAP
Consistenty (linearizability), Availability,Partition tolerance 
المهم احنا محتاجين نضيف Request of data فى اكتر من داتا ستور طبعا بعد ال formating وليكن الريكوست هيجى من API json وبعدين حسب كل نوع داتا محتاجين نضيف الريكوست دا
جايز يكون شكل الحل عامل كدا ويب سيرفس هضيف هتكلم كل ال data stores سواء serial  او parallel جايز يكون دا حل سهل لكن عرضة لمشاكل وهى ال inconsistency وكمان ال race condition لو عندى concurrent user ولو حصل أنه ضاف البيانات فى داتا ستور وفشل فى التانى وكمان لو فيه عندى integrity 

![Diagram](/public/blog-1-1.jpg)

زى بتشيك على unique value قبل ما أضيف الخ فهضطر انى هيكون عندى تاسك صعبة وهى انى ادور على ال confilect دا واحله بنفسى ، طيب ماذا لو عملنا committed log زى فكرة ال WAL فى ال single leader replication عندى log بكتب فيه وبعد كدا كل داتا ستور له consumer بيقرأ ال log
ومحتفظ بال current position عشان يعرف هو واقف فين فى ال log فكل consumer هيقرا الداتا ويكتبها بال order إلى انكتب بيه على ال log وال web API هتكتب فى ال log بشكل append only without mutation هل كدا حلينا مشاكلنا لأ احنا حلينا مشكلة ال race condition بس 
![Diagram](/public/blog-1-2.jpg)
ايه إلى هيحصل لو consumer وقع دى حلها بردوا فى ال log وهو كل consumer زى ما قولنا هيحتفظ بال position الى كان واقف عنده فى reliable storage ولما يقوم تانى هيفضل يسحب لحد ال current position والى هو بشكل نظرى لو بطلنا نكتب فى ال log هيكون consistent ودا eventual consistenty
طيب فيه عندنا مشاكل تانية تخيل أن عندنا data integrity تخيل أن عندنا ال user name مثلا unique ومحتاج اتأكد منه قبل ما يحصل مشكلة جايز يكون الحل فى انى كل consumer يتأكد هل اليوزر دا موجود ولا لأ لكن للاسف الحل دا هيسبب لنا مشكلة race condition تانى لو فيه concurrent user
المشكلة دى بتتحل ببساطة فى ال monolithic هى انى اضيف unique constraint على ال field لكن الحل دا مش متاح فى حالة زى دى ، المشكلة دى ممكن احلها بانى اعمل event stream ال stream مش هيضمن ليا هنا ال unique لكن هيدخلى الداتا in serial order و هيمنع فرصة concurrency
بعد كدا اقدر اعمل single thread stream processor وظيفته أنه يتأكد من ال data integrity قبل ما يضيف new record in db فكدا حلينا مشكلة ال data integrity لكن للاسف الحل دا لحاله كدا مش scalable لان عشان نضمن أن مفيش race condition خلينا فيه single thread stream processor
فالداتا هتتأخر كتير ، المشكلة دى ممكن احلها عن طريق استخدام partitioned committed log زى مثلا Kafka او اى committed log تانى يكون بيدعم ال partitioning لكن ولأن ال ordering مقدرش اضمنها الا على مستوى ال partition الواحد فهنا لازم اعمل partition key
يكون بيضمن أن الداتا المتطابقة إلى مش عايزها تتكرر تدخل فى نفس البارتشن زى مثلا أن ال key بتاعى يكون ال user name دا مثلا فانا كدا اقدر ازود partition as I need واقدر اقوم اكتر من consumer واكون حليت مشكلة ال scalability طبعا الحل دا له مشاكل بردوا زى ال hotspot partition
وهو ممكن تلاقى بارتشن عليه زحمة اكتر من بارتشن تانى لكن دا مش موضوع الثريد نتكلم عنها فى ثريد تانى أن شاء الله
Mohammed Ragab
Mohammed Ragab
@rgbdev
Backend / DevOps engineer
Follow on Twitter
twitter-thread.com/t/1601656457443028993
Missing some tweets in this thread? Or failed to load images or videos? You can try to force a refresh.

