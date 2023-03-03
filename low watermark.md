
كنا اتكلمنا عن الhigh watermark خلينا نتكلم عن الlow watermark.
هو ايه المشكلة الى عندنا فى الWAL بنفضل نعملlog لكل العلميات الى بتم على الpersistent store والى طبع ممكن يكبر مع مرور الوقت طبعا فيه باترن تانى وهو اsegmented log الى بيخلينى اتعامل مع فايلات صغيرة
بدل ما اتعامل مع فايل واحد كبير ودا مستخدم فى حاجات كتير زى كافكا ممكن نتكلم عنه فى ثريد منفصل لان له مشكلة تانية بيحلها لكن حتى مع كدا الdisk نفسه ممكن يكبر مع مرور الوقت طيب ايه الحل الحل هو انى امسح بعض الlogs بشكل مستمر طيب همسح ايه وازاى لازم الاقى طريقة اقول بيهاان الجزء دا من الLog اقدر امسحه بشكل امن عن طريق انى اجيب الlowest offset او الlower watermark اذن الlow watermark هوindex فى الwal بيحدد انهى جزء من البيانات اقدر امسحه وانا مطمن طيب دا هيحصل ازاى والسيستم شغال عن طريق انى بعملtask شغالة فى الbackground علىthread منفصلشغالة بشكل مستمر بتشوف ايه الجزء من الLog الى ينفع امسحه طيب هتحدد دا بناءا على ايه فيه طريقتين

1- snapshot based low watermark

الطريقة دى اغلب الconsensus بتستخدمها زىRAFT , Zookeeper وهنا الstorage engine بياخدsnapshot بشكل دورى مش بس كدا كمان بياخد الlog indexالى حصل لهapply بشكل سليم اول ما الsnapshot يحصل لها تخزين على الdisk بشكل سليم اصبحتdurable هنا الlog manager بيحط الlow watermark عشان يبدأ يمسح الLogs القديمة كل الLogs segments بيجيب اخرentry فيها لو اقل من الsnapshot index بيmark الsegment دى عشان تتمسح

2- Time based low watermark

فى الحالة دى لما بيكون الlog مش بالضرورة مستخدم عشانsystem state update هنا اقدر اعملdelete لبعض الlogs الى عدى عليها وقت معين بدون ما انتظر بقية الsub systems انها تبعت الlowest log زى فى حالة الtime retain policy فىKafka topicsالsegments الى اخرentry فيها عدى عليها(7 ايام مثلا داdefault فى كافكا) بيمسحها ودا عن طريق ان كلlog entry بيكون لهtimestamp الى حصل creation فيه
