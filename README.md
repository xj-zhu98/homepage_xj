# Multidata Lab

Welcome to the Multidata Research Group !

## Table Of Contents
  * [Getting started](#getting-started)
  * [Main pages](#main-pages)
    + [Home](#home)
      - [News](#news)
      - [Selected Publications](#selected-publications)
    + [Team](#team)
    + [Publications](#publications)
    + [Research](#research)
    + [Join Us](#join-us)
  * [License](#license)

## Getting started

Want to learn more about Jekyll? Check out [this tutorial](https://www.taniarascia.com/make-a-static-website-with-jekyll/).
Why Jekyll? Read [Andrej Karpathy's blog post](https://karpathy.github.io/2014/07/01/switching-to-jekyll/)!

## Main pages

网站导航栏页面，可在`_pages/`目录下按需求新增。网站总配置文件`_config.yml`修改title、脚注和基础配置等。

### Home

Home页面概要位于`_pages/about.md`，排版设置位于`_layouts/about.html`.

#### News

更新Home页面的News显示，可在`_news/`目录下新增md文件。目前显示条数限制为3条，可在`_layouts/about.html`进行修改。

#### Selected Publications

更新Home页面的Selected Publications显示，可在`_bibliography/papers.bib`文献配置文件中，设置`selected  = {true}`即可。

### Team

Team页面概要位于`_pages/team.md`，排版设置位于`_layouts/team.html`. 

更新Team页面的成员信息，可进入`_data/group_members.yml`添加或修改成员信息，例如：
```yaml
- category: Master  
  people:
    - name: Xinjun Zhu
      image: /assets/img/team_photo/zxj_zju.jpg  # 将个人照片存放至`/assets/img/team_photo/`目录下
      desc: Master (Start in 2021).
      url: /team/xinjun-zhu/  # 个人主页链接，可以放置自己的外部个人网站，也可在`_pages/member_page/`目录下创建md文件制作自己的个人主页
      email: xjzhu@zju.edu.cn
```

### Publications

Publications页面概要位于`_pages/publications.md`，排版设置位于`_layouts/bib.html`. 

更新Publications论文，可进入`_bibliography/papers.bib`添加和修改文献信息，例如：
```yaml
# 会议论文格式
@inproceedings{SIGIR23_KRDN,       
  author       = {Xinjun Zhu and
                  Yuntao Du and
                  Lu Chen and
                  Baihua Zheng and
                  Yunjun Gao},
  title        = {Knowledge-refined Denoising Network for Robust Recommendation},
  booktitle    = {The 46th International {ACM} {SIGIR} Conference on Research
                  and Development in Information Retrieval (SIGIR)},
  year         = {2023},
  pdf          = {https://arxiv.org/pdf/2304.14987.pdf},  # 论文pdf链接
  code         = {https://github.com/xj-zhu98/KRDN},  # 论文代码链接
  selected     = {true},    # 是否在Home页面的Selected Publications栏目显示
  abbr         = {SIGIR},   # 会议名称
  preview      = {SIGIR23_KRDN.jpg}  # 论文框架图，放置于`/assets/img/publication_preview/`目录下
}

# 期刊论文格式
@article{TKDE23_Model,
  author    = {Minjun Zhao and
               Lu Chen and
               Keyu Yang and
               Yuntao Du and 
               Yunjun Gao},
  title     = {Finding Materialized Models for Model Reuse},
  journal   = {IEEE Transactions on Knowledge and Data Engineering (TKDE)},
  year      = {2023},
  abbr      = {TKDE}
}
```

### Research

Research页面概要位于`_pages/research.md`，直接在该页面进行添加和修改操作。

### Join Us

Join Us页面概要位于`_pages/join_us.md`，直接在该页面进行添加和修改操作。

## License

Powered by <a href="https://jekyllrb.com/" target="_blank">Jekyll</a> with <a href="https://github.com/alshedivat/al-folio">al-folio</a> theme.

The theme is available as open source under the terms of the [MIT License](https://github.com/alshedivat/al-folio/blob/master/LICENSE).
