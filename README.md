一、Masonry相关

1.maker

[view mini_makeConstraints:^(BDMiniVConstraintMaker *v, BDMiniHConstraintMaker *h) {
    v.top.equalTo(superview.mini_top).with.offset(0);     
    v.bottom.equalTo(superview.mini_bottom).with.offset(10);
    
    h.left.equalTo(superview.mini_left).with.offset(-10);
    h.right.equalTo(superview.mini_right).with.offset(0);
}];

BDMiniLayout会自动调用view.translatesAutoresizingMaskIntoConstraints=NO;

2.equalTo、lessThanOrEqualTo、greaterThanOrEqualTo

.equalTo -- NSLayoutRelationEqual
.lessThanOrEqualTo -- NSLayoutRelationLessThanOrEqual
.greaterThanOrEqualTo -- NSLayoutRelationGreaterThanOrEqual

3.Attribute

BDMiniViewAttribute	NSLayoutAttribute
view.mini_left	                NSLayoutAttributeLeft
view.mini_right	                NSLayoutAttributeRight
view.mini_top	                NSLayoutAttributeTop
view.mini_bottom	        NSLayoutAttributeBottom
view.mini_leading	        NSLayoutAttributeLeading
view.mini_trailing	        NSLayoutAttributeTrailing
view.mini_width	        NSLayoutAttributeWidth
view.mini_height	        NSLayoutAttributeHeight
view.mini_centerX	        NSLayoutAttributeCenterX
view.mini_centerY	        NSLayoutAttributeCenterY
view.mini_baseline	        NSLayoutAttributeBaseline

3.with

v.width.equalTo(superview.width).with.offset(10);

h.left.greaterThanOrEqualTo(label.mini_left).with.priorityLow();
v.top.equalTo(label.mini_top).with.priority(600);

4.and

h.left.and.right.equalTo(superview);

5.offset的正负

v.top.equalTo(superview.top).offset(-10);

6.NSArray

v.height.equalTo(@[view1.mini_height, view2.mini_height]);
v.height.equalTo(@[view1, view2]);

h.left.equalTo(@[view1, @100, view3.right]);

7.Prioritize（优先级）

.priority 指定优先级
.priorityHigh -- UILayoutPriorityDefaultHigh
.priorityMedium -- 在low和hight之间优先级
.priorityLow -- UILayoutPriorityDefaultLow

8.Composition

一般情况下，不推荐使用，因为区分Vertical和Horizontal更加容易维护。

1）edges

v.edges.equalTo(view2);
v.edges.equalTo(superview).insets(UIEdgeInsetsMake(5, 10, 15, 20))

2）size

v.size.greaterThanOrEqualTo(titleLabel)

3）center

v.center.equalTo(superview).centerOffset(CGPointMake(-5, 10))
v.center.equalTo(button1)


9.References

@property (nonatomic, strong) MASConstraint *topConstraint;

[view mini_makeConstraints:^(BDMiniVConstraintMaker *v, BDMiniHConstraintMaker *h) {
    self.topConstraint = v.top.equalTo(superview.mini_top).with.offset(10);
    h.left.equalTo(superview.mini_left).with.offset(5);
}];

[self.topConstraint uninstall];

10.约束最好写在updateConstraints中，记得在末尾调用[super updateConstraints]，并设置translatesAutoresizingMaskIntoConstraints是否为YES。

二、Visual Format Language（VFL）

1.Apple官方文档

https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage.html

2.语法

Visual Format Language语法	
1）<visualFormatString>	  (<orientation>:)? (<superview><connection>)? <view>(<connection><view>)* (<connection><superview>)?

2）<orientation>	        H|V 

3）<superview>         	  | 

4）<view>	                [<viewName>(<predicateListWithParens>)?] 

5）<connection>	          e|-<predicateList>-|- 

6）<predicateList>	      <simplePredicate>|<predicateListWithParens>

7）<simplePredicate>	    <metricName>|<positiveNumber>

8）<predicateListWithParens>	(<predicate>(,<predicate>)*) 

9）<predicate>	          (<relation>)?(<objectOfPredicate>)(@<priority>)? 

10）<relation>	          ==|<=|>= 

11）<objectOfPredicate>	  <constant>|<viewName> (see note) 

12）<priority>	          <metricName>|<number>

13）<constant>	          <metricName>|<number>

14）<viewName>	          Parsed as a C identifier. This must be a key mapping to an instance of NSView in the passed views dictionary. 

15）<metricName>	        Parsed as a C identifier. This must be a key mapping to an instance of NSNumber in the passed metrics dictionary. 

16）<number>		          As parsed by strtod_l, with the C locale. 


3.封装API

/*
 *根据约束使其在superView中生效
 *需要区分版本！！
 *@superView     需要添加约束的View
 *@constraints   约束NSArray
 */
+ (void)applyConstraintToView:(UIView *)superView
                  constraints:(NSArray<__kindof NSLayoutConstraint *> *)constraints;


/*
 * !!!注意：
 * 1.会自动将viewsDic中的View的translatesAutoresizingMaskIntoConstraints属性设置为NO，
 * 除了superView；
 * 2.约束的数值只能int和float，float不要加.f结尾。正常的如10, 0.5, 10.5
 *
 *根据VFL语法字符串参数使其在superView中生效，同时支持参数。
 *@superView     需要添加约束的View
 *@strArray      VFL语法NString
 *@viewsDic      VFL中的UIView字典
 *@option        NSLayoutFormatOptions选项
 */
+ (void)applyConstraintByStringsAndOption:(UIView *)superView
                                 strArray:(NSArray<__kindof NSString * > *)strArray
                                 viewsDic:(NSDictionary *)viewsDic
                                   option:(NSLayoutFormatOptions)option;

3.NSLayoutFormatOptions

typedef NS_OPTIONS(NSUInteger, NSLayoutFormatOptions) {
    NSLayoutFormatAlignAllLeft = (1 << NSLayoutAttributeLeft),
    NSLayoutFormatAlignAllRight = (1 << NSLayoutAttributeRight),
    NSLayoutFormatAlignAllTop = (1 << NSLayoutAttributeTop),
    NSLayoutFormatAlignAllBottom = (1 << NSLayoutAttributeBottom),
    NSLayoutFormatAlignAllLeading = (1 << NSLayoutAttributeLeading),
    NSLayoutFormatAlignAllTrailing = (1 << NSLayoutAttributeTrailing),
    NSLayoutFormatAlignAllCenterX = (1 << NSLayoutAttributeCenterX),
    NSLayoutFormatAlignAllCenterY = (1 << NSLayoutAttributeCenterY),
    NSLayoutFormatAlignAllBaseline = (1 << NSLayoutAttributeBaseline),
    NSLayoutFormatAlignAllLastBaseline = NSLayoutFormatAlignAllBaseline,
    NSLayoutFormatAlignAllFirstBaseline NS_ENUM_AVAILABLE_IOS(8_0) = (1 << NSLayoutAttributeFirstBaseline),
    
    NSLayoutFormatAlignmentMask = 0xFFFF,
    
    NSLayoutFormatDirectionLeadingToTrailing = 0 << 16, // default
    NSLayoutFormatDirectionLeftToRight = 1 << 16,
    NSLayoutFormatDirectionRightToLeft = 2 << 16,  
    
    NSLayoutFormatDirectionMask = 0x3 << 16,  
};

//设置两个UILabel和右边iconImageView水平方向的间距以及垂直居中对齐
[NSLayoutConstraint constraintsWithVisualFormat:@"H:|-15-[label1]-0-[label2]-0-[iconImageView(22)]-15-|"
                                            options:NSLayoutFormatAlignAllCenterY
                                            metrics:nil
                                              views:viewsDictionary];


//注意：如果前面最后一个|（看起来与[superView]等价）改为[superView]，是否可以省去这行呢？
//答案是不行的，这就是VFL的诡异之处！
//我们需要单独写VFL，来设置右边iconImageView与superView垂直居中对齐
[constraints addObjectsFromArray:[NSLayoutConstraint constraintsWithVisualFormat:@"H:[iconImageView]-(<=1)-[superView]"
                                            options:NSLayoutFormatAlignAllCenterY
                                            metrics:nil
                                              views:viewsDictionary]];
//到此，一个水平方向上的两个UILabel和一个iconImageView垂直居中对齐设置完成了。

4.VFL使用注意事项

1）将手动添加constraint的UIView设置属性translatesAutoresizingMaskIntoConstraints为NO；
2）明确每个UIView的位置、大小；
3）@250、@750、@1000代表优先级，见前面语法，经常需要优先级配合完成constraint的设置；
4）当设置一个UIView的宽度、高度的时候，如果不想与superView有间距关系，则-(<=1)-即可；
5）[UIView addConstraints:constraints]要被废除，用 [NSLayoutConstraint activateConstraints:constraints]代替；
6）@优先级，1000是代表必须满足，所以如果某个UIView会根据业务动态隐藏或者显示，则应该设置为<1000的优先级，这样当这个UIView的hidden属性为YES的时候，其相关constraint会被别取代；
7）constraint的添加必须在addSubView调用之后。
