1. 起因： 项目中经常用到弹框，可是项目中并没有好用的基于vue1.0的弹框组件，于是自己写了一个

2. 前言： 由于本人水平有限，公司业务需要，半年前从java转为前端开发，代码写得比较简陋，如有谬误，请勘正务究！

3. 用法示例：
    ```javascript
    <template>
    <dialog :dialog.sync="dialog" :fullfilled="fullfilled" v-if="dialog" v-ref:dialog>
        <div v-if="dialog.type > 3">
            <div>自定义模板html</div>
        </div>
    </dialog>
    </template>
    <script>
        import Dialog from 'component/common/dialog'
    
        export default {
            methods: {
                doSomething() {
                    // 实例化并弹出
                    if (ids.length > 0) {
                        this.dialog = {
                            type: 4,
                            title: '违规下架',
                            submitCb: this.shelveItem,
                            submitParams: ids
                        };
                        this.quantity = ids.length;
                        this.invalid = false;
                        this.reason = '';
                        this.reasonHint = '';
                    } else {
                        this.dialog = {
                            type: 1
                        }
                    }
                }
            }
        }
    </script>

    ```

4. 参数说明：
4.1 dialog对象为实例化属性配置对象，配置弹出框的大小，标题，内容等，具体含义请参照源码注释
4.2 mask配置是否需要蒙板
4.3 fullfilled 用于回调校验不成功阻止关闭，在弹框中给出校验失败信息
4.4 如果想自定义默认的初始化参数，可以通过config配置，以定义内置type=1,2,3时的大小和内容
