# `f1` Laying Out Fields In Forms

{{ toc }}

## Usage

In simple forms (login form), add fields to Form's `views` property:

    'form' => Form::new([
        ...
        'views' => [
            'title' => Input::new([
                ...
                // sort inside the parent property
                'sort_order' => 10,
                
                // on wide screen, use half of the width
                'wrap_modifier' => '-narrow',
            ]),
        ],
    ]),

In complex forms (product editing form), define form tabs and sections and add fields to `views` property of a tab or a section: 

    'form' => Form::new([
        ...
        'tabs' => [
            'general' => Tab::new([
                ...
                'views' => [
                    'title' => Input::new([...]),
                    'pricing' => Section::new([
                        'views' => [
                            'price  ' => Input::new([...]),
                        ],
                    ]),
                ],
            ]),
        ],
    ]),

## Under The Hood

A single field container (form, tab or section) renders fields as follows:

    <div class="... form-fields">
        <div class="form-fields__wrap">
            <div class="input"></div>
        </div>
        <div class="form-fields__wrap -narrow">
            <div class="input"></div>
        </div>
        ...
    </div>
            
`.form-fields__wrap -narrow` are full-width by default, and there is a resize script bound to `form-fields` which make them 50% width if the container is wide enough.

## Why This Way

It follows "`f2` Implementing View Containers".