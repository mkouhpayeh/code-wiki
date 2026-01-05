# CSS Tips

---

## List and Item

```
.obj-container {
    width: 100%;
    padding: 55px 0;
    padding: 3.4375rem 0rem;
    box-sizing: border-box;
}

.obj-container .obj-item {
    border-top: 1px solid #ccc;
    padding: 35px 15px;
    padding: 2.1875rem .9375rem
}

.obj-container .obj-item.is-project .obj-item {
    border-bottom: 0;
    padding: 0 0 20px;
    padding: 0rem 0rem 1.25rem
}

.obj-container .obj-item:first-child {
    border-top: none;
    padding: 0 15px 35px;
    padding: 0rem .9375rem 2.1875rem;
}

@media (max-width: 576px) {
    .obj-container .obj-item {
        width:100%
    }
}

.side-navigation {
    padding: 55px 0 0;
    padding: 3.4375rem 0rem 0rem;
}

.side-navigation .category-holder{
    width: 100%;
    margin-bottom: 60px;
    margin-bottom: 3.75rem;
    -webkit-user-select: none;
    user-select: none;
}

.side-navigation .category-holder .category-title {
    color: #828282;
    margin-bottom: 15px;
    margin-bottom: .9375rem;
}

.side-navigation .category-holder .category-item {
    -webkit-transition: color .3s ease;
    -moz-transition: color .3s ease;
    -ms-transition: color .3s ease;
    -o-transition: color .3s ease;
    transition: color .3s ease;
    width: 100%;
    cursor: pointer;
    margin-bottom: 15px;
    margin-bottom: .9375rem
}

/* --------------- */
/* Alignment       */
/* --------------- */

.justify-content-start {
    -ms-flex-pack: start!important;
    justify-content: flex-start!important
}

.justify-content-end {
    -ms-flex-pack: end!important;
    justify-content: flex-end!important
}

.justify-content-center {
    -ms-flex-pack: center!important;
    justify-content: center!important
}

.justify-content-between {
    -ms-flex-pack: justify!important;
    justify-content: space-between!important
}

.justify-content-around {
    -ms-flex-pack: distribute!important;
    justify-content: space-around!important
}

.align-items-start {
    -ms-flex-align: start!important;
    align-items: flex-start!important
}

.align-items-end {
    -ms-flex-align: end!important;
    align-items: flex-end!important
}

.align-items-center {
    -ms-flex-align: center!important;
    align-items: center!important
}

.align-text-center {
    text-align: center
}

.align-items-baseline {
    -ms-flex-align: baseline!important;
    align-items: baseline!important
}

.align-items-stretch {
    -ms-flex-align: stretch!important;
    align-items: stretch!important
}

.align-content-start {
    -ms-flex-line-pack: start!important;
    align-content: flex-start!important
}

.align-content-end {
    -ms-flex-line-pack: end!important;
    align-content: flex-end!important
}

.align-content-center {
    -ms-flex-line-pack: center!important;
    align-content: center!important
}

.align-content-between {
    -ms-flex-line-pack: justify!important;
    align-content: space-between!important
}

.align-content-around {
    -ms-flex-line-pack: distribute!important;
    align-content: space-around!important
}

.align-content-stretch {
    -ms-flex-line-pack: stretch!important;
    align-content: stretch!important
}

.align-self-auto {
    -ms-flex-item-align: auto!important;
    align-self: auto!important
}

.align-self-start {
    -ms-flex-item-align: start!important;
    align-self: flex-start!important
}

.align-self-end {
    -ms-flex-item-align: end!important;
    align-self: flex-end!important
}

.align-self-center {
    -ms-flex-item-align: center!important;
    align-self: center!important
}

.align-self-baseline {
    -ms-flex-item-align: baseline!important;
    align-self: baseline!important
}

.align-self-stretch {
    -ms-flex-item-align: stretch!important;
    align-self: stretch!important
}

```

```
<div class="row">
    <div class="col-lg-2 hide-xs show-l">
        <div class="side-navigation">
            <div class="category-holder">
                <div class="category-title">
                    <div>
                        cat
                    </div>
                </div>
                <div class="category-item">
                    <div>
                        item
                    </div>
                </div>
                <div class="category-item">
                    <div>
                        item
                    </div>
                </div>
            </div>
        </div>
    </div>
    <div class="col-12 col-md-12 col-lg-10">
        <div class="obj-container">
            <div class="obj-list">
                <div class="obj-item">
                    <div class="row">
                        <div class="col-12 col-sm-4 col-md-3 col-lg-2">
                            <p>Item1</p>
                            <p>ddddd</p>
                        </div>
                        <div class="col-12 col-sm-4 col-md-4 col-lg-6">
                            xxx1
                        </div>
                        <div class="col-12 col-sm-4 col-md-5 col-lg-4 align-self-center">
                            xxx1
                        </div>
                    </div>
                </div>
                <div class="obj-item">
                    <div class="row">
                        <div class="col-12 col-sm-4 col-md-3 col-lg-2">
                            <p>Item2</p>
                            <p>ddddd</p>
                        </div>
                        <div class="col-12 col-sm-4 col-md-4 col-lg-6">
                            xxx2
                        </div>
                        <div class="col-12 col-sm-4 col-md-5 col-lg-4 align-self-center">
                            xxx2
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```
