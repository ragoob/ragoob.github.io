فى ال podcast عن كافكا اتكلمنا عن مفهوم فى ال **replication** وهو ال **High water mark**

فالثريد دى هتكلم شوية عن المفهوم دا فخلينى اتكلم شوية عن المشكلة هو احنا ايه المشكلة الى بنحاول نعالجها .

فى ال **write a head log**

ودا باترن بنستخدمه عشان نعمل recovery لل consistent state بعد ما السيرفر يرجع من ال crash ممكن تقرأ شوية تفاصيل عنه فى الثريد دى <https://t.co/r8QCCryusA> 

ال **write-ahead log** مش كفاية عشان تدينا availability لما السيرفر يقع

لأن زى ما عارفين ان لو عندنا سيرفر واحد ووقع خلاص الكلاينتس مش هيقدروا يعملوا حاجة تجاه السيرفر دا ولا يتكلموا معاه لحد ما السيرفر دا يرجع تانى لكن احنا ممكن نحلها عن طريق اننا ن replicate ال log

على اكتر من سيرفر عن طريق ال leader -followers

الداتا من الكلاينت بتروح لليدر وبعدين الليدر هي replicate ال log entries لمجموعة من ال follower.

(**Quorum**)

تقدر تفهم ال **leader – followers** من السلسلة دى لأحمد الإمام <https://t.co/5zLlzACVSV> 

طيب لحد دلوقتى مفيش مشكلة واضحة طيب الليدر دا هيقع اكيد فى وقت ما مش هيفضل شغال من غير مشاكل هنا بيجى عملية ال **leader election** واحد من ال follower دلوقتى محتاج يمسك مكانه عشان يحافظ على ال availability عشان الكلاينت يفضل شغال زى ما كان بشكل طبيعى لكن عندنا فى الحتة دى مشكلتين

- الليدر ممكن يقع قبل ما يبعت كل ال log entries لأى followers
- ` `الليدر ممكن يقع قبل ما يبعت البيانات لأغلب ال followers او Quorum فممكن يكون فيه followers عندها بيانات اكتر من الباقى او كلهم فقدوا البيانات دى log entries هنا بيجى دور ال  high-water mark 

طيب دا بيحصل ازاى بئا

- كل log entry الليدر بيبدأ انه يضيفه ل local log عنده وبعد كدا يبعته لل followers
- ال followers بتهندل ال replication request وهتضيف ال entry ل ال local log بتاعها

- كل followers بتبعت رد لل leader بال index بتاع اخر log entry عندهم

وكمان بتبعت ال Generation Clock ودا موضوع تانى ممكن نتكلم عنه فى ثريد منفصله لانه بيحل مشكلة تانية بردوا

- الليدر بيفضل متابع ال indexes الى حصل لها replication لكل followers

- ال **high-water mark** ممكن نعرفها ازاى اننا نبص فى ال log indexes بتاعت ال followers وكمان الليدر

وناخد الاندكس الى متاحة فى مجموعة من السيرفرات   **Quorum** 

- ` `الليدر بيبعت **high water mark** لل followers بشكل مستمر ممكن يكون عن طريق انه يبعت **heart beats** او فى ال replication request او request منفصل مش هتفرق كتير اى entry دون ال **high-water mark** مفيش كلاينت يقدر يقرأه لان زى ما قولنا عشان نتفادى المشاكل فى حالة ال leader election

هل كدا المشكلة اتحلت لأ لسه عندنا مشكلة لما سيرفر يدخل او يرجع من ال crashing ممكن يكون عنده   Log entries conflict 

فهنا هو محتاج يسأل ال leader عن ال **high water mark** الأول وبعدين يشوف ايه ال entries الى عاملة conflict او الى ممكن تعمل وبعدين يعمل truncate ل ال logs لحد النقطة الى بت match

وبعد كدا يبدأ ياخد ال entries من ال leaders عشان يتأكد انه متفق مع باقى ال cluster

كل ال **Consensus algorithms** بتستخدم الكونسيبت دا

**RAFT, ZAB etc.**

