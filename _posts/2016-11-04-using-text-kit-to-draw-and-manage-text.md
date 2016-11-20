---
layout: post
title: "使用Text Kit来绘制和管理文本"
date: 2016-11-04 08:29:59 +0800
categories:
---

引言：本文来自Apple官方文档《Text Programming Guide for iOS: Using Text Kit to Draw and Manager》，可能会由于本人英语水平较低（未过英语四级考试）、理解能力和文档时效等问题造成与原文出入较大，阅读时请慎重！如遇到问题，请在评论中指出。

---

正如《Displaying Text Content in iOS》部分描述，UIKit 框架中提供了一些用来在App的用户界面中展示文本的类：UITextView, UITextField, UILabel 和 UIWebView。由 UITextView 类创建的文本视图意在用来展示大量的文本。UITextView 底层有一个非常强大的布局引擎叫做 Text Kit。如果你需要自定义或者介入这个布局过程，你可以使用 Text Kit。对于更细粒度的文本和特殊的需要自定义解决方案的需求，可以使用《Lower Level Text-handling Technologies》中描述的更底层的技术作为替代。

TextKit 是 UIKit 框架中提供的高品质的保存、布局、展示精细排版的所有特征（如字距、连字、换行和对齐）的排版服务的类和协议的集合。TextKit 构建于 CoreText 之上，所以它提供了同样的速度与强大。UITextView 完全集成了 Text Kit，它提供的编辑和展示功能允许用户输入文本、指定格式属性并查看结果。Text Kit 其他方法提供了文本存储和布局功能。图1展示了 Text Kit 在 iOS 文本和图像框架中的位置。

图1 Text Kit框架位置
![Text Kit Framework Position]({{ site.baseurl }}/images{{ page.id }}/1.png)

Text Kit 给了你在用户界面元素上渲染文本的完整的控制。除 UITextView 外，UITextField 和 UILabel 也是在 Text Kit 之上构建的，它可以无缝地和动画、UICollectionView 和 UITableView 集成。Text Kit 被设计为拥有完全可扩展的面向对象架构，支持子类化、委托和一组完整的通知，支持深度定制。

## TextKit主要的成员对象

Text Kit 主要对象之间的数据流图如图2。Text views 是 UITextView 类的实例，Text containers 是 NSTextContainer类的实例，layout manager 是 NSLayoutManager 类的实例，Text storage 是 NSTextStorage 类的实例。在 Text Kit 中，一个 NSTextStorage 对象存储用于 UITextView 对象显示和 NSLayoutManager 对象在一个 NSTextContainer 对象上布局的文本。

图2 主要的Text Kit对象
![Primary Text Kit Objects]({{ site.baseurl }}/images{{ page.id }}/2.png)

一个 NSTextContainer 对象可以定义一个可供文本布局的区域。text container 通常会定义一个方形区域，但可以通过创建一个 NSTextContainer 的子类来定义其他形状如圆、五边形或不规则的形状。text container 不仅定义了可以填充文本区域的边界，同时它还维护了一个定义区域中不能布局文本的地方（例外区）的 Bezier paths 数组。当布局发生时，文本在例外区以外布局，这就为包含图像和其他非文本元素提供了一种方式。

NSTextStorage 为 Text Kit 的扩展文本处理系统（extended text-handling system）定义了基本的存储机制。NSTextStorage 是 NSMutableAttributedString 的一个子类，储存了文本系统操作的文本和属性。它保证了文本和属性可以在编辑操作中保持一贯的状态。除了保存文本，NSTextStorage 对象还管理了一些 NSLayoutManager 对象的集合，在必要时将文本或属性的变化通知给它们来重新布局或重新展示。

NSLayoutManager 对象统筹其他文本处理对象的操作。协调将 NSTextStorage 中的数据转换为在视图的显示区域中渲染的文本的操作。将 Unicode 码映射为字符并监督 NSTextContainer 对象定义的区域上的布局。

> 注意： NSLayoutManager、NSTextStorage 和 NSTextContainer 可以在子线程中访问，但要保证只在单一的线程中被访问。

## 文本属性

Text Kit 处理三种文本属性：字符属性、段落属性和文档属性。字符属性包括如字体、颜色和下标等可以在单独的字符或一个区间的字符关联的特性。段落属性是如缩进、tabs和行间距等的特性。文档属性包括如纸张大小，边距和视图缩放比例等文档级别的特性。

### 字符属性

属性字符串（attributed string）用字典中的键值对来存储字符属性。键作为属性的名称，用一个字符串常量标识符（如NSFontAttributeName）来表示。图3展示了一个属性字典作用于一个属性字符串中的一个区间上。

图3 属性字符串的组成
![Composition of an attributed string]({{ site.baseurl }}/images{{ page.id }}/3.png)

理论上，属性字符串中每个字符都有一个相关联的属性字典。然而通常一个属性字典作用于更长的字符区间上。NSAttributedString 类提供了根据字符的位序获取关联的属性字典和该属性字典的属性值作用的区间的方法如：`attributesAtIndex:effectiveRange:`.

除了使用预定义的属性外，你可以选择向一个字符区间指定任意的属性键值对。你可以向 NSTextStorage 中合适的字符区间通过 NSMutableAttributedString 的`addAttribute:value:range:`方法添加属性，你也可以创建一个包含一些自定义属性的键和值的字典然后通过`addAttributes:range:`方法一步将它们添加到一个字符区间上。要使用自定义的属性时，你需要自定义一个的 NSLayoutManager 的子类来使用它们。你自定义的子类应该重写`drawGlyphsForGlyphRange:atPoint:`方法。在这个方法里可以先调用父类相应的方法来绘制字符区间然后在上面加上你自定义的属性，或者也完全使用自己的方式绘制字符。

### 段落属性

段落属性影响布局管理器（layout manager）将文本行布置到页面航的段落中的方式。文本系统在 NSParagraphStyle 类的对象中封装段落属性。预定义的字符属性中的 NSParagraphStyleAttributeName 键的值指向一个包含了这个字符区间的段落属性的 NSParagraphStyle 对象。属性修复（attribute fixing）确保只有一个 NSParagraphStyle 对象属于各个段落中的字符。

段落属性包含如对其、制表符停止（tab stop）、换行模式、和行间距（也被称为leading）等特性。

### 文档属性

文档属性属于文档整体。文档属性包含纸张大小、边距和视图缩放比例等特性。尽管文本系统中没有内置的存储文档属性的机制，NSAttributedString 的初始化方法如`initWithRTF:documentAttributes:`可以将从 RTF 或 HTML 数据流中派生出的文档属性填充到你提供的占位字典对象中。相反的，如果给写入RTF数据的方法如`RTFFromRange:documentAttributes:`传递一个包含文档属性的字典，它会写入文档属性。

### 属性修正

编辑属性字符串可能引起的前后不一致必须通过属性修复（attribute fixing）来清除。UIKit 通过定义`fixAttributesInRange:`方法扩展了 NSMutableAttributedString 来修复附件、字符和段落属性中的矛盾。这些方法保证了当附件字符被删除时其所关联的属性也将不复存在，字符属性只应用在支持这个字体的字符上，段落属性在整个段落中保持一致。

## 使用编程的方式改变文本存储

NSTextStorage 对象为 Text Kit 提供字符仓库的服务。此数据的格式是字符序列（使用unicode编码）和关联了属性（如字体、颜色和段落属性）的属性字符串。表示属性字符串的类有 NSAttributedString 和 NSMutableAttributedString（NSTextStorage 就是它的子类）。如字符属性小节描述，每一个文本块中的字符都有一个键值对字典与其相关联。键为属性名（如NSFontAttributeName），对应的值指定字符的这个属性的特征（如Helvetica 12-point）。

通过编程的方式编辑text storage对象有三个阶段。第一个阶段是向其发送一个`beginEditing`消息来宣布一组修改。

在第二个阶段中，你向它发送一些编辑消息如`replaceCharactersInRange:withString:`和`setAttributes:range:`来使字符或属性改变。每次发送这样的消息时，text storage 对象调用`edited:range:changeInLength:`来跟踪从收到`beginEditing`消息后受影响的字符区间。

在第三个阶段中，当你完成了对 text storage 对象的修改，向其发送一个`endEditing`消息。这会使其调用其 delegate 的`textStorage:willProcessEditing:range:changeInLength:`和其自身的`processEditing`方法，修复记录的有修改的字符区间上的属性。

修复完属性后，text storage 对象向其 delegate 发送`textStorage:didProcessEditing:range:changeInLength:`方法，给 delegate 一个验证并进行可能的修改的时机。（尽管 delegate 可以在这个方法中修改 text storage 对象的字符属性，但不可以在 text storage 对象处于冲突状态时修改字符本身。）最后，text storage 对象向每个关联的 layout manager 发送`processEditingForTextStorage:edited:range:changeInLength:invalidateRange:`消息--指出 text storage 对象中改变了的字符区间以及这些修改的性质。layout manager 们轮流的使用这些信息来重新计算他们的文字位置并在必要时重新展示。

## 使用字体对象

计算机字体是使用了如 OpenType 或 TrueType 的格式的数据文件，包含描述一组字形的信息，以及在字形渲染中使用的各种补充信息。UIFont 类提供了获取和设置字体信息的接口。一个 UIFont 实例提供对字体特性和字形的访问。Text Kit 结合字符信息和字体信息来选择文本布局时要使用的字形。通过将字体对象传递给接收字体作为参数的方法中来使用字体对象。字体对象是不可变的，所以可以安全地在应用中多个线程中使用它们。

要使用`preferredFontForTextStyle:`或`fontWithName:size:`来创建 UIFont对象而不能使用 alloc 和 init 方法来创建。也可以用 font descriptor 通过`fontWithDescriptor:size:`来创建字体。这些方法会先从现有的字体对象中查找是否存在指定特征的字体对象，如果有就返回它，否则，根据请求的字体数据创建一个合适的字体对象。

### 文本类型

文本类型（Text styles），iOS 7 带来的由被称为动态类型(*Dynamic Type*)的机制实现的对字体的预期用途的语义描述。文本类型通过 UIFontDescriptor.h 中定义的常量（如下表所示）进行组织使用。用于由文本样式描述的目的的实际字体可能基于一些动态考虑而变化，包括由 UIApplication 属性`preferredContentSizeCategory`表示的用户的内容大小类别偏好。要获取给定文本样式的字体对象，您需要将相应的常量传递给 UIFont 方法`preferredFontForTextStyle:`. 要获取文本样式的字体描述符，请将常量传递给 UIFontDescriptor 的`preferredFontDescriptorWithTextStyle:`方法。

表1：文本类型常量

---------------------------|------------------------
UIFontTextStyleHeadline    |    用于标题的字体
UIFontTextStyleSubheadline |    用于子标题的字体
UIFontTextStyleBody        |    用于正文文本的字体
UIFontTextStyleFootnote    |    用于注脚的字体
UIFontTextStyleCaption1    |    用于标准字幕的字体
UIFontTextStyleCaption2    |    用于替代字幕的字体

文本类型通过动态类型机制为应用带来了许多提高文本可读性的优势，动态类型以协调的方式响应用户的偏好，并响应辅助功能设置以增强易读性和超大类型。也就是说，当你调用`preferredFontForTextStyle:`时，返回的特定的字体除了对指定的文本类型常量用途进行处理外，还包括根据用户偏好和上下文而变化的特征，包括跟踪（字母间距）调整。

根据文本类型常量得到的字体应该被用于应用中所有的文本中，除了界面元素如按钮、标签和条（bar）等。当然你需要合适你应用的文本类型。同时，观察`UIContentSizeCategoryDidChangeNotification`也很重要，这样可以在用户修改了内容尺寸类别时重新布局。当你的应用收到这个通知时，它应该向 Auto Layout 布局的 view 发送`invalidateIntrinsicContentSize`或者向手动布局的用户界面元素发送`setNeedsLayout`消息。同时要使先前的 font 或 font descriptor 失效，然后获取新的。

### 使用字体描述

字体描述，UIFontDescriptor类的实例，提供了通过一个属性字典来描述字体的方式，并可以用它来创建 UIFont 对象。你可以从一个字体描述创建一个 UIFont 对象，也可以从 UIFont 对象得到一个描述，还可以改变这个描述然后用它生成一个新的字体对象。你也可以用一个字体描述来指定应用中提供的自定义的字体。

字体描述可以被存档，这样可以高效的使用文本类型。你不应该为文本类型缓存字体对象，因为这是动态的（根据用户的偏好不同会改变）。但你可以缓存字体的描述，然后稍后用它再创建相同特征的字体对象。

你可以使用字体描述来查询系统中符合特定属性的可用字体，然后创建符合这些属性（名称、特点、语言或其他特性）的字体对象。例如，你可用使用一个字体描述来获取所有的符合给定的字体簇名称的字体，使用CSS标准定义的簇名称：

```
UIFontDescriptor *helveticaNeueFamily = [UIFontDescriptor fontDescriptorWithFontAttributes:@{UIFontDescriptorFamilyAttribute: @“Helvetica Neue”}];
NSArray *matches = [helvetecaNeueFamily matchingFontDescriptorsWithMandatoryKeys:nil];
```

`matchingFontDescriptorsWithMandatoryKeys:`方法返回系统中所有的 Helvetica Neue 字体的描述的数组，如HelveticaNeue, HelveticaNeue-Medium, HelveticaNeue-Light, HelveticaNeue-Thin,等。

你可以通过应用字形特征（如粗体、斜体、宽松、紧凑）来修改`preferredFontForTextStyle:`方法返回的字体，可以使用字体描述来修改特定的特征：

```
UIFontDescriptor *fontDescriptor = [UIFontDescriptor preferredFontDescriptorWithTextStyle:UIFontTextStyleBody];
UIFontDescriptor *boldFontDescriptor = [fontDescriptor fontDescriptorWithSymbolicTraits:UIFontDescriptorTraitBold];
UIFont *boldFont = [UIFont fontWithDescriptor:boldFontDescriptor size:0.0];
```

这个代码片段先取到正文文本类型的字体描述，然后修改这个字体描述为加粗特征，最后使用 UIFont 类的`fontWithDescriptor:size:`方法返回一个实际用于正文文本的加粗字体对象。传递给`fontWithDescriptor:size:`方法的size为0.0是为了保护字体描述返回的原始尺寸属性。由于字体大小是由动态类型机制决定的，所以提倡这么做。

### 激活字体特征

字体描述的另一个重要的用途就是激活和选择自体特征。 字体特征是当字体由文本系统呈现时控制其外观的方面的印刷属性。字体特征只在字体设计者选择包含字体特性的字体中可用。有一些字体特征只在少许的字体中可用，而更多的特征在大多数字体中可用。另外，同一字体的不同版本，安装在不同平台上，它们可用的字体特征可能是不同的。

字体特征分组为称为特征类型的类别，其中各个特征选择器选择特定特征设置。 要素类型可以是排他的或非排他的。 如果要素类型是排他的，则一次只能选择一个可用的要素选择器，例如数字是比例还是固定宽度。 如果要素类型是非专有的，则可以一次启用任意数量的要素选择器。 例如，对于连字特征类型，您可以选择字体支持的可用连字类别的任意组合。

> 注意，如果选择字体中不可用的功能，则不会看到字体字形外观的更改。

一些特征是上下文的，而另一些特征是非上下文的。 将上下文特征应用于字形的方式取决于与相邻字形相比的字形位置。 文本系统布局功能的一个强大的方面是它能够自动应用复杂的上下文处理。

非上下文特征以相同的方式应用于字形，而不管相邻的字形。 这些功能包括为了数学排版或增强排版复杂性而选择交替字形集以给予文本不同的外观和字形替代。

示例：激活 Helvetica Neue Medium 字体定义的两个特征类型：

```
NSArray *timeFeatureSettings = @[
     @{
          UIFontFeatureTypeIdentifierKey: @(kNumberSpacingType),
          UIFontFeatureSelectorIdentifierKey: @(kProportionalNumbersSelector)
     },
     @{
          UIFontFeatureTypeIdentifierKey: @(kCharacterAlertnativesType),
          UIFontFeatureSelectorIdentifierKey: @(2)
     }];
UIFont *originalFont = [NSFont fontWithName:@“HelveticaNeue-Medium” size:12.0];
UIFontDescriptor *originalDescriptor = [originalFont fontDescriptor];
UIFontDescriptor *timeDescriptor = [originalDescriptor fontDescriptorByAddingAttributes: @{UIFontDescriptorFeatureSettingsAttribute:timeFeatureSettings}];
UIFont *timeFont = [UIFont fontWithDescriptor:timeDescriptor size:12.0];
```

上述清单中的代码激活数字间距要素类型（由常量kNumberSpacingType表示），选择比例宽度数字（kProportionalNumbersSelector）和字符替换要素类型（kCharacterAlternativesType），其中要素选择器标识符键值为2。 本示例中用于表示字体要素类型和选择器的常量在Core Text框架（CoreText / CoreText.h）中的头文件SFNTLayoutTypes.h中声明为枚举。 在字符替换类型的情况下，没有预定义的常量来表示特征选择符标识符，因此您只需使用字体定义的数字值。

因为字体特征是由字体定义的，所以确定支持的特征的最可靠的方法是直接查询字体。 您可以使用Core Text中的CTFontCopyFeatures函数来完成此操作，如下述清单所示。

```
UIFont *font = [UIFont fontWithName:@“HelveticaNeue-Medium” size:12.0];
CFArrayRef fontFeatures = CTFontCopyFeatures((__bridge CTFontRef)font);
NSLog(@“properties = %@, fontFeatures);
```

下面的清单中展示了上述清单中的`CTFontCopyFeatures`函数的结果，即控制台日志：

```
properties = (
        {
        CTFeatureTypeExclusive = 1;
        CTFeatureTypeIdentifier = 6;
        CTFeatureTypeName = "Number Spacing";
        CTFeatureTypeNameID = 266;
        CTFeatureTypeSelectors =         (
                        {
                CTFeatureSelectorDefault = 1;
                CTFeatureSelectorIdentifier = 0;
                CTFeatureSelectorName = "No Change";
                CTFeatureSelectorNameID = 264;
            },
                        {
                CTFeatureSelectorIdentifier = 1;
                CTFeatureSelectorName = "Proportional Numbers";
                CTFeatureSelectorNameID = 267;
            }
        );
    },
        {
        CTFeatureTypeExclusive = 1;
        CTFeatureTypeIdentifier = 17;
        CTFeatureTypeName = "Character Alternatives";
        CTFeatureTypeNameID = 262;
        CTFeatureTypeSelectors =         (
                        {
                CTFeatureSelectorDefault = 1;
                CTFeatureSelectorIdentifier = 0;
                CTFeatureSelectorName = "No Change";
                CTFeatureSelectorNameID = 264;
            },
                        {
                CTFeatureSelectorIdentifier = 1;
                CTFeatureSelectorName = "Alternate Punctuation";
                CTFeatureSelectorNameID = 263;
            },
                        {
                CTFeatureSelectorIdentifier = 2;
                CTFeatureSelectorName = "Numbers Punctuation";
                CTFeatureSelectorNameID = 265;
            }
        );
    }
)
```

在这种情况下，结果表明，此版本的 Helvetica Neue Medium 字体有两个字体特征：数字间距和字符替换。 当使用字体描述符激活字体特征并在其设置中进行选择时，此结果中最重要的值是特征类型标识符和特征选择符标识符。 将这些值添加到表示字体特征设置的字典数组中，并将该数组用作`UIFontDescriptorFeatureSettingsAttribute`的值，依次传递给`fontDescriptorByAddingAttributes:`或`fontDescriptorWithFontAttributes:`方法。 该列表中显示的常量的枚举值与`CTFontCopyFeatures`函数返回的要素类型标识符和要素选择器标识符的数值相关联。

可以看到，从CTFontCopyFeatures函数产生的字体特征数组还显示要素类型是否是排它的，以及哪个要素选择器是默认的。 并且，当然，要素类型名称和要素选择器名称值为可用的字体要素及其设置提供人类可读的标识。

### 查询字体度量

UIFont定义了一些方法来访问字体的度量信息，当该信息可用时。 诸如ascender，capHeight，xHeight等属性都对应于标准字体度量信息。 图4显示了字体指标如何适用于字形尺寸，表2列出了与各种指标相关的属性名称。 有关更多具体信息，请参阅属性描述。

图4 字体度量
![Font metrics]({{ site.baseurl }}/images{{ page.id }}/4.png)

表2 字体度量和UIFont方法联系

----------------|---------------
: Font metric : | : Properties :
X-height        | xHeight
Ascent          | ascender
Cap height      | capHeight
Line height     | lineHeight
Descent         | descender
Point size      | pointSize

## 布局文本

布局管理器对象（NSLayoutManager类的实例），是 Text Kit 中文本展示的核心控制对象，布局管理器执行以下的操作：

* 控制 text storage 和 text container 对象
* 根据字符生成字形
* 计算字形位置并存储该信息
* 管理字形和字符的区间
* 文本视图（text view）请求时在文本视图上绘制文字
* 计算文本行的边界
* 控制连字
* 操作字符和字形属性

在MVC范式中，布局管理器充当了controller。NSMutableAttributedString 的子类 NSTextStorage 提供 Model 的一部分功能，负责字符串及其属性，如字体、样式、颜色、大小。NSTextContainer 也可以被认为是 Model 的一部分，因为它对放置文本的页面的几何布局进行建模。UITextView（或其他UIView对象）提供文本展示的View。NSLayoutManager用作文本系统的控制器，因为它将文本存储对象中的字符翻译为字形，根据一个或多个文本容器对象的尺寸将它们排成行，并将文本显示在一个或多个文本视图对象中进行协调。

## 布局过程

布局管理器在分两步执行文本布局：字形生成和字形布局。 布局管理器根据需要懒惰地执行这两个布局步骤。 因此，一些 NSLayoutManager 方法触发字形生成，而其他的不会，字形布局也是如此。 在它生成字形、计算布局位置之后，布局管理器缓存这些信息以提高后续调用的性能。

布局管理器缓存字形，属性和布局信息。 它跟踪已通过更改文本存储中的字符而失效的字形范围。 有两种方式可以自动使字符范围失效：如果它需要生成字形或需要字形布局。 如果愿意，您可以手动使字形或布局信息无效。 当布局管理器接收到需要知道在无效范围中的字形或布局的消息时，它根据需要生成字形或重新计算布局。

### 生成行片段边界

布局管理器在 NSTextContainer 对象内以字形行布置文本。 文本容器内的这些行的布局由其形状和其包含的任何排除路径确定。 在行片段矩形与由排除路径定义的区域相交的部分中的行必须缩短或分段; 如果在整个区域上存在间隙，则将与其重叠的行必须移位以补偿。

布局管理器为给定行提出一个矩形，然后要求文本容器调整矩形到合适的位置。 所提出的矩形通常跨越文本容器的边界矩形，但是它可以更窄或更宽，甚至它也可以部分地或完全地位于边界矩形之外。 布局管理器发送文本容器以调整建议的矩形的消息是`lineFragmentRectForProposedRect:atIndex:writingDirection:remainingRect:`，它根据文本布局的方向返回相对建议矩形可用的最大矩形。它还返回一个包含任何剩余空间的矩形，例如在文本容器中的另一侧留下的的孔或间隙。

布局管理器在将文本实际匹配到矩形中时会进行一个最终调整。此调整是由文本容器固定的小量，称为行片段填充，其定义行片段矩形的每一端上的部分留空。文本在行片段矩形内插入该量（矩形本身不受影响）。填充允许在文本容器的区域的边缘（和任何孔周围）小规模调整，并且保持文本不直接邻接在该区域附近显示的任何其它图形。您可以使用`lineFragmentPadding`属性从其默认值更改填充。请注意，行片段填充不是表示边距的合适手段。对于文档边距，应在UITextView对象的封装视图中设置其位置和大小。对于文本边距，应该设置文本视图的`textContainerInset`属性。此外，您可以使用`NSMutableParagraphStyle`属性（如headIndent）为单个段落设置缩进值。

除了返回线片段矩形本身之外，布局管理器返回称为使用的矩形的矩形。 这是行片段矩形的实际包含要绘制的字形或其他标记的部分。 按照惯例，两个矩形包括行片段填充和行间间隔（根据字体的行高度量和段落的行间距参数计算）。 但是，段落间距（前后）和文本周围添加的任何空间（例如由中心间隔文本引起的空间）仅包含在行片段长方形中，并且不包括在使用的矩形中。

### 指定排除路径

文本容器维护一个UIBezierPath对象数组，表示其边界矩形内的排除路径。 当布局管理器发送文本容器`lineFragmentRectForProposedRect:atIndex：writingDirection:remainingRect:`消息提出与由排除路径定义的区域之一相交的行片段矩形时，文本容器返回排除该区域的调整后的行片段矩形。 此过程如图5所示。

图5 行片段适应
![Line fragment fitting]({{ site.baseurl }}/images{{ page.id }}/5.png)

### 指定多页和多列布局

在最简单的情况下，Text Kit对象被单一配置，即一个文本存储对象，一个文本容器和一个布局管理器，如图6所示。 在 Interface Builder 中从对象库中拖动文本视图时，将自动实例化此配置。 UITextView 对象提供其他对象并将它们连接在一起。 您还可以在代码中创建此安排，如清单6所示。

图6 
![Object configuration for a single text flow]({{ site.baseurl }}/images{{ page.id }}/6.png)

您还可以在代码中创建此安排，如清单6所示。 此代码可以在视图控制器中，例如，UIViewController 的子类，其有名为 textContainer 的 NSTextContainer 属性。

```
// 清单6
NSTextStorage *textStorage = [[NSTextStorage alloc]initWithString:string];
NSLayoutManager *layoutManager = [[NSLayoutManager alloc]init];
[textStorage addLayoutManager:layoutManager];
self.textContainer = [[NSTextContainer alloc]initWithSize:self.view.bounds.size];
[layoutManager addTextContainer:self.textContainer];
UITextView *textView = [[UITextView alloc]initWithFrame:self.view.bounds textContainer:self.textContainer];
[self.view addSubview:textView];
```

此配置受限于只有一个文本容器和一个文本视图。 在这种布置中，文本流在由文本容器限定的区域内不间断。分页符，多列布局和更复杂的布局不能由这种安排满足。

通过使用多个文本容器，每个具有相关联的文本视图，更复杂的布局布置是可能的。 例如，为了支持分页符，应用程序可以配置文本对象如图7所示。

图7
![Object configuration for paginated text]({{ site.baseurl }}/images{{ page.id }}/7.png)

每个文本容器对应于文档的一页。 显示文本的视图可以嵌入你应用提供的自定义视图对象中，作为文本视图的背景。此自定义视图反过来可以嵌入在UIScrollView对象中，以使用户能够滚动浏览文档的页面。

多列文档可以使用类似的对象排列进行建模，如图8所示。

图8
![Object configuration for multicolumn text]({{ site.baseurl }}/images{{ page.id }}/8.png)

如果不是有一个文本容器对应于单个页面，而是有两个文本容器 -- 一个页面上的每一列。 每个文本容器控制文档的一部分。 在显示文本时，字形首先布局在左上角的容器中。 当该视图中没有更多的空间时，布局管理器通知其代理它已经完成填充容器。 代理可以检查是否有更多的文本需要布局和添加另一个文本容器如果必要。 布局管理器继续在下一个容器中布置文本，完成后通知代理，等等。 再次，自定义视图（描绘为蓝色矩形）为这些文本列提供了一个画布。

不仅可以有多个文本容器，还可以有多个 NSLayoutManager 对象访问同一个文本存储。 图9说明了具有多个布局管理器的对象排列。 该布置的效果是为相同文本提供的多个视图。 如果用户改变顶视图中的文本，则该改变立即反映在底视图中（假设改变的位置在底视图的界限内）。

图9
![Object configuration for multiple views of the same text]({{ site.baseurl }}/images{{ page.id }}/9.png)
