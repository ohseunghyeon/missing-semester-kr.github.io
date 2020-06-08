---
layout: page
title: 여러분의 CS 교과과정에서 빠진 학기
---

CS 교과과정은 운영체제에서 기계학습에 이르기까지 CS 내의 고급 주제에 대해 모두 가르쳐 주지만, 거의 다루지 않는 중요한 한 가지 과목이 있으며, 대신 학생들이 스스로 알아내야 하는 도구 숙련도가 있다. 우리는 당신에게 command line을 마스터하는 방법, 강력한 text editor 사용, version control system의 화려한 기능 사용 방법 등을 가르쳐 줄 것입니다!

학생들은 교육 과정 동안 이러한 도구들을 사용하여 수백 시간을 보내므로, 가능한 한 유연하고 마찰이 없는 경험을 하는 것이 이치에 맞습니다. 이러한 도구들을 마스터하게 되면 자신의 의지에 맞게 도구를 알맞게 사용하는 방법을 알아내는 데 시간을 덜 쓸 수 있을 뿐만 아니라 이전에 불가능할 정도로 복잡해 보였던 문제들을 해결할 수 있습니다.


이 [수업의 동기](/about/)에 대해 읽어보세요.

{% comment %}
# Registration

Sign up for the IAP 2020 class by filling out this [registration form](https://forms.gle/TD1KnwCSV52qexVt9).
{% endcomment %}

# 시간표

{% comment %}
**Lecture**: 35-225, 2pm--3pm<br>
**Office hours**: 32-G9 lounge, 3pm--4pm (every day, right after lecture)
{% endcomment %}

<ul>
{% assign lectures = site['2020'] | sort: 'date' %}
{% for lecture in lectures %}
    {% if lecture.phony != true %}
        <li>
        <strong>{{ lecture.date | date: '%-m/%d' }}</strong>:
        {% if lecture.ready %}
            <a href="{{ lecture.url }}">{{ lecture.title }}</a>
        {% else %}
            {{ lecture.title }} {% if lecture.noclass %}[no class]{% endif %}
        {% endif %}
        </li>
    {% endif %}
{% endfor %}
</ul>

녹화된 강의 영상은 [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J)에서 볼 수 있습니다.

# 수업 내용

**Staff**: 이 수업은 [Anish](https://www.anishathalye.com/), [Jon](https://thesquareplanet.com/), 그리고 [Jose](http://josejg.com/)가 함께 가르칩니다.
**Questions**: [missing-semester@mit.edu](mailto:missing-semester@mit.edu)로 Email을 보내주시면 됩니다.

# Beyond MIT

또한 우리는 다른 이들이 이러 자료로부터 이익을 얻을 수 있기 바라면서 MIT를 뿐만 아니라 외부와 이 수업을 공유해 왔습니다. 이 수업에 대한 게시물과 토론은 아래와 같습니다:

 - [Hacker News](https://news.ycombinator.com/item?id=22226380)
 - [Lobsters](https://lobste.rs/s/ti1k98/missing_semester_your_cs_education_mit)
 - [/r/learnprogramming](https://www.reddit.com/r/learnprogramming/comments/eyagda/the_missing_semester_of_your_cs_education_mit/)
 - [/r/programming](https://www.reddit.com/r/programming/comments/eyagcd/the_missing_semester_of_your_cs_education_mit/)
 - [Twitter](https://twitter.com/jonhoo/status/1224383452591509507)
 - [YouTube](https://www.youtube.com/playlist?list=PLyzOVJj3bHQuloKGG59rS43e29ro7I57J)

# 번역

- [English: https://missing.csail.mit.edu/](https://missing.csail.mit.edu/)
- [中文: missing-semester-cn.github.io](https://missing-semester-cn.github.io/)
- [Türk: missing-semester-tr.github.io](https://missing-semester-tr.github.io/)
- [한국어: missing-semester-kr.github.io](https://missing-semester-kr.github.io/)

참고: 이 링크는 커뮤니티 번역에 대한 외부 링크 입니다. 번역에 대해 따로 심사하지 않았습니다.

이 수업의 코스 노트를 번역했나요? 그렇다면 목록에 추가할 수 있도록 [PR](https://github.com/missing-semester/missing-semester/pulls)을 제출해주세요!


## Acknowledgements

이 강의의 비디오를 녹화할 수 있게 도와준 Elaine Mello, Jim Cain, [MIT Open Learning](https://openlearning.mit.edu/); A/V 장비에 도움을 준 Anthony Zolnik과 [MIT AeroAstro](https://aeroastro.mit.edu/); 그리고 이 수업을 지지해 주신 Brandi Adams와 [MIT EECS](https://www.eecs.mit.edu/)에 감사드립니다.

---

<div class="small center">
<p><a href="https://github.com/missing-semester/missing-semester">Source code</a>.</p>
<p>Licensed under CC BY-NC-SA.</p>
<p>See <a href="/license/">here</a> for contribution &amp; translation guidelines.</p>
</div>
