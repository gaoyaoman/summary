# 前端分享

组件封装基本原则是：`高内聚，低耦合，易读写，可复用`。一般在开发中组件可分为`业务组件`和`公共组件`。

## 业务组件封装示例

这里举广西建院的习题库功能需求为🌰。广西建院的习题库功能中包含一个`教师可以管理、编辑考试题和查看学生答题结果，学生可以答题并查看答题结果`的这么个需求。
从这个需求中可以分析得出，这是个`试卷组件`,且试卷组件中包含`预览、编辑、答题和答案对比`的功能。抽象一下这些功能，我们可以把以上的功能归类为
`预览模式、编辑模式、答题模式和校对模式`四种模式。那显然我们的试卷组件就有一个`mode`的`props`属性:

```javascript
    //组件模式
    // EXAMINATION_PREVIEW 习题预览模式
    // EXAMINATION_TEST    测试模式
    // EXAMINATION_VERIFY  校验模式
    'mode': {
        type: String,
        required: true,
        default: "EXAMINATION_PREVIEW"
    }            
```

在father组件中调用时就可以根据不同的业务模式传入不同的mode属性值就可以控制试卷组件的业务模式。

```javascript
    <examination
        mode="EXAMINATION_TEST"
    >
    </examination>                    
```

同样的，对于考试卷的具体业务需求和功能实现的必须数据，可相对应的增加其他`props`属性：

```javascript
        props: {
            //只读模式
            'readonly':{
                type: Boolean,
                required: true,
                default: false
            },
            //组件模式的容器高度
            'tableHeight':{
                type: String,
                default: 'calc(100vh - 260px)'
            },
            //试卷id
            'examinationId':{
                type: Number,
                required: true,
                default: 0
            },
            //组件模式
            // EXAMINATION_PREVIEW 习题预览模式
            // EXAMINATION_TEST    测试模式
            // EXAMINATION_VERIFY  校验模式
            'mode':{
                type: String,
                required: true,
                default: "EXAMINATION_PREVIEW"
            },
            //校验模式的源数据
            'verifyData': {
                type: Array,
                default: () => new Array()
            }
        }
```

father组件调用如下：

```javascript
    <!-- 2.学生做题 -->
    <examination
        v-if="componentName === 'doExercise'"
        mode="EXAMINATION_TEST"
        @submit="handleSubmitExamination"
        @navigate-back="navigateBack"
        :examination-id="examinationID"
        :readonly="false">
    </examination>
```

>考试组件的内容其实就是题目，抽象一下就是一个一个的题目组件生成的考试卷，那我们就可以把题目抽象为一个单独的`试题组件`:


```javascript
        props: {
           //题目类型
           'questionType': {
               type: String,
           },
           //题干
           'questionContent':{
               type: String,
           },
           //答案选项
           'answerItems': {
               type: Array,
           },
           //题目的id
           'questionId':{
               type: Number,
               default: 0
           },
           //学生的答案
           'studentAnswer': {
               type: Array,
           },
           //模式
           // EXAMINATION_PREVIEW 习题预览模式
           // EXAMINATION_TEST    测试模式
           // EXAMINATION_VERIFY  校验模式
           'mode': {
               type: String,
               default: 'EXAMINATION_PREVIEW'
           }
       }
```


>我们通过遍历试卷的`configData`来生成整张试卷：


```javascript
        <div :class="readonly ? 'examination__body examination__body--readonly' : 'examination__body'"
             :loading="loading">
            <div class="question__type-container"
                 v-for="(questionTypeData, questionTypeIndex) in questionData">
                <span class="question__type">
                    {{ questionTypeData['Type'] }}
                </span>
                <div :class="mode === 'EXAMINATION_PREVIEW' ? 'question' : 'question question--no-line'"
                     v-for="(questionItem, questionItemIndex) in questionTypeData['Subjects']">
                    <div class="question__body">
                        <examination-question
                            @change="handleAnswerChange"
                            :mode="mode"
                            :question-id="questionItem['ID']"
                            :question-type="questionItem['Type']"
                            :question-content="`${questionItemIndex + 1} ${questionItem['Subject']}`"
                            :student-answer="questionItem['StudentAnswer'] || []"
                            :answer-items="questionItem['Answers']">
                        </examination-question>
                    </div>
                    <div class="question__operation">
                        <i title="删除"
                           v-if="!readonly && mode !== 'EXAMINATION_TEST'"
                           @click="handleRemoveQuestion(questionItem)"
                           class="iconfont iconfont-in-table">
                            &#xe62c;
                        </i>
                    </div>
                </div>
            </div>
        </div>
```

于是，我们就可以通过以上的方式得到满足功能需求的一个试卷组件。

>预览模式

![预览模式](images/预览模式.png)

>编辑模式

![编辑模式](images/编辑模式.png)

>考试模式

![考试模式](images/考试模式.png)

>校验模式

![校验模式](images/校验模式.png)
