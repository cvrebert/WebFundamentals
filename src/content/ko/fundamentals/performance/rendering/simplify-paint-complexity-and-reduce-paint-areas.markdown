---
title: "그림 그리기 복잡성 단순화 및 그림 그리기 영역 줄이기"
description: "그림 그리기(paint)는 최종적으로 사용자의 화면에 합성되는 픽셀을 채우는 과정입니다. 대체로 파이프라인의 모든 작업 중 가장 오래 실행되는 과정으로 가급적 피해야 합니다."
updated_on: 2015-03-20
notes:
  highdpi:
    - "높은 DPI 화면에서 위치가 고정된 요소는 자체 컴포지터 레이어로 자동 승격됩니다. 이는 낮은 DPI 장치에는 적용되지 않습니다. 승격이 텍스트 렌더링을 하위 픽셀에서 회색조로 변경하고 레이어 승격이 수동으로 수행되어야 하기 때문입니다."

key-takeaways:
  - 변형 또는 불투명도를 제외한 모든 속성 변경은 항상 그림 그리기를 트리거합니다.
  - 그림 그리기는 종종 픽셀 파이프라인에서 가장 비용이 많이 드는 부분이므로 가급적 피하십시오.
  - 레이어 승격 및 애니메이션 오케스트레이션(orchestration)을 통해 그림 그리기 영역을 줄이십시오.
  - Chrome DevTools 그림 그리기 프로파일러를 사용하여 그림 그리기 복잡성과 비용을 평가하고 가능하면 이를 줄이십시오.


---
<p class="intro">
  그림 그리기(paint)는 최종적으로 사용자의 화면에 합성되는 픽셀을 채우는 과정입니다. 대체로 파이프라인의 모든 작업 중 가장 오래 실행되는 과정으로 가급적 피해야 합니다.
</p>

{% include shared/takeaway.liquid list=page.key-takeaways %}

요소의 기하학적 형태 변경은 픽셀 수정을 의미하므로 레이아웃을 트리거하는 경우 _항상 그림 그리기를 트리거_하게 됩니다!

<img src="images/simplify-paint-complexity-and-reduce-paint-areas/frame.jpg" class="g--centered" alt="전체 픽셀 파이프라인">

또한 배경, 텍스트 색상 또는 그림자와 같은 비기하학적 속성을 변경하는 경우에도 그림 그리기를 트리거할 수 있습니다. 이러한 경우에 레이아웃이 필요하지 않으며 파이프라인의 모양은 다음과 유사합니다.

<img src="images/simplify-paint-complexity-and-reduce-paint-areas/frame-no-layout.jpg" class="g--centered" alt="레이아웃이 없는 픽셀 파이프라인">

## Chrome DevTools를 사용하여 빠르게 그림 그리기 병목 현상 식별

Chrome DevTools를 사용하여 그림 그리기 중인 영역을 빠르게 식별할 수 있습니다. DevTools로 이동하여 키보드의 Esc 키를 누릅니다. 표시되는 패널의 Rendering 탭으로 이동하여 "Show paint rectangles"를 선택합니다.

<img src="images/simplify-paint-complexity-and-reduce-paint-areas/show-paint-rectangles.jpg" class="g--centered" alt="DevTools의 Show paint rectangles 옵션">

이 옵션이 켜져 있으면 그림 그리기가 발생할 때마다 Chrome의 화면이 녹색으로 깜박입니다. 전체 화면이 녹색으로 깜박이거나 생각하지 않았던 화면 영역을 그림 그려야 하는 경우에는 약간 더 연구해야 합니다.

<img src="images/simplify-paint-complexity-and-reduce-paint-areas/show-paint-rectangles-green.jpg" class="g--centered" alt="그림 그리기가 발생할 때마다 녹색으로 깜박이는 페이지">

Chrome DevTools Timeline에 추가 정보를 제공하는 그림 그리기 프로파일러 옵션이 있습니다. 이 옵션을 활성화하려면 Timeline으로 이동하여 상단에서 “Paint” 상자를 선택합니다. 이 옵션은 오버헤드를 수반하고 성능 프로파일링을 왜곡하므로, _그림 그리기 문제를 프로파일링하려는 경우에만 이 옵션을 켜십시오_. 그림 그리기 중인 항목을 정확히 파악하고자 하는 경우에 사용하는 것이 가장 좋습니다.

<img src="images/simplify-paint-complexity-and-reduce-paint-areas/paint-profiler-toggle.jpg" class="g--centered" alt="Chrome DevTools에서 그림 그리기 프로파일링을 활성화로 전환">

이제 여기에서 Timeline 레코딩을 실행할 수 있고 그림 그리기 레코드가 더욱 상세히 수행됩니다. 프레임에서 그림 그리기 레코드를 클릭하면 해당 프레임의 그림 그리기 프로파일러에 액세스할 수 있습니다.

<img src="images/simplify-paint-complexity-and-reduce-paint-areas/paint-profiler-button.jpg" class="g--centered" alt="그림 그리기 프로파일러를 표시하는 버튼">

그림 그리기 프로파일러를 클릭하면 그림 그려진 항목, 걸린 시간 및 필요했던 개별 그림 그리기 호출을 볼 수 있는 보기가 표시됩니다.

<img src="images/simplify-paint-complexity-and-reduce-paint-areas/paint-profiler.jpg" class="g--centered" alt="Chrome DevTools 그림 그리기 프로파일러">

이 프로파일러를 통해 영역과 복잡성(실제로 그림 그리기에 걸린 시간)을 모두 알 수 있습니다. 이들은 모두 그림 그리기를 피할 수 없는 경우 수정할 수 있는 영역입니다.

## 이동 또는 페이드 요소 승격

그림 그리기가 항상 메모리에 단일 이미지로 수행되는 것은 아닙니다. 실제로 필요에 따라 브라우저가 다중 이미지 또는 컴포지터 레이어로 그림을 그릴 수 있습니다.

<img src="images/simplify-paint-complexity-and-reduce-paint-areas/layers.jpg" class="g--centered" alt="컴포지터 레이어 표시">

이 접근방식의 이점은 정기적으로 다시 그림 그리거나 변형으로 화면에서 움직이는 요소를 다른 요소에 영향을 주지 않고 처리할 수 있다는 것입니다. 이는 Sketch, GIMP 또는 Photoshop과 같은 아트 패키지와 동일합니다. 서로의 최상위에서 개별 레이어를 처리하고 합성하여 최종 이미지를 생성할 수 있습니다.

새 레이어를 생성하는 가장 좋은 방법은 `will-change` CSS 속성을 사용하는 것입니다. 이 속성은 Chrome, Opera 및 Firefox에서 작동하며 `transform` 값으로 새 컴포지터 레이어를 생성합니다.

{% highlight css %}
.moving-element {
  will-change: transform;
}
{% endhighlight %}

Safari나 Mobile Safari처럼 `will-change`를 지원하지 않지만 레이어 생성을 활용하는 브라우저의 경우, 3D 변형을 사용(오용)하여 새 레이어를 강제 적용해야 합니다.

{% highlight css %}
.moving-element {
  transform: translateZ(0);
}
{% endhighlight %}

하지만 각 레이어는 메모리와 관리가 필요하므로 너무 많은 레이어를 생성하지 않도록 주의하십시오. 이에 대한 자세한 내용은 [컴포지터(compositor) 전용 속성 고수 및 레이어 수 관리](stick-to-compositor-only-properties-and-manage-layer-count) 절을 참조하십시오.

요소를 새 레이어로 승격시킨 경우 DevTools를 사용하여 그렇게 하는 것이 성능상의 이점이 있는지 확인합니다. **프로파일링 없이 요소를 승격하지 마십시오.**

## 그림 그리기 영역 줄이기

하지만 때로는 요소 승격에도 불구하고 그림 그리기 작업이 계속 필요합니다. 그림 그리기의 중대한 문제는 브라우저가 그림 그리기가 필요한 두 영역을 통합하고 전체 화면을 다시 그림 그릴 수 있다는 것입니다. 따라서 예를 들어 페이지 상단에 고정된 헤더가 있는 경우, 화면 하단에서 무언가의 그림을 그리면 전체 화면이 다시 그림 그릴 수 있습니다.

{% include shared/remember.liquid title="Note" list=page.notes.highdpi %}

애니메이션 및 전환을 많이 겹치지 않도록 오케스트레이션하거나 페이지의 특정 부분의 애니메이션을 피하는 방법으로 그림 그리기 영역을 줄일 수 있습니다.

## 그림 그리기 복잡성 단순화
그림 그리기는 작업에 따라 비용에 차이가 있습니다. 예를 들어, 그림자처럼 흐리게 만드는 효과 적용은 빨간색 상자를 그리는 것보다 시간이 더 많이 소요됩니다. 하지만 CSS 관점에서 보면 이것이 항상 명확한 것은 아닙니다. `background: red;` 및 `box-shadow: 0, 4px, 4px, rgba(0,0,0,0.5);`는 반드시 매우 다른 성능 특성을 가진 듯이 보일 필요가 없지만 그렇게 보입니다.

<img src="images/simplify-paint-complexity-and-reduce-paint-areas/profiler-chart.jpg" class="g--centered" alt="화면 일부를 그림 그리는 데 걸리는 시간">

위의 그림 그리기 프로파일러를 사용하면 효과를 달성하기 위해 다른 방법을 살펴볼 필요가 있는지 판별할 수 있습니다. 가장 비용이 적게 드는 스타일 집합이나 대체 수단을 사용하여 최종 결과를 얻을 수 있는지 살펴보십시오.

특히 애니메이션 동안 항상 그림 그리기를 피하려는 경우 일반적으로 프레임당 **10ms**는 특히 모바일 장치에서 그림 그리기 작업을 수행하기에 충분하지 않습니다.


