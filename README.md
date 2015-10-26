
<link href="<?php echo app()->request->baseUrl;?>/themes/indition/css/select2.min.css" rel="stylesheet" type="text/css" s>
<script src="<?php echo app()->request->baseUrl;?>/themes/indition/js/select2.min.js" type="text/javascript"></script>

<div class="form" style="margin-top: 15px;">


    <?php $form=$this->beginWidget('CActiveForm', array(
        'id'=>'sales-form',
        'enableAjaxValidation'=>true,
        'enableClientValidation'=>true,
        'clientOptions'=>array('validateOnSubmit' => true),
        'htmlOptions'=>array('class'=>'form-horizontal')
    ));
    echo $form->hiddenField($model, 'id');
    ?>

    <p class="note">Fields with <span class="required">*</span> are required.</p>

    <div class="control-group">
        <?php echo $form->labelEx($model,'first_name',array('class'=>'control-label')); ?>
        <div class="controls">
            <?php echo $form->textField($model,'first_name',array('maxlength'=>250, 'style' => 'width: 359px;')); ?>
            <?php echo $form->error($model,'first_name'); ?>
        </div>
    </div>
    <div class="control-group">
        <?php echo $form->labelEx($model,'last_name',array('class'=>'control-label')); ?>
        <div class="controls">
            <?php echo $form->textField($model,'last_name',array('maxlength'=>250, 'style' => 'width: 359px;'));?>
            <?php echo $form->error($model,'last_name'); ?>
        </div>
    </div>
    <div class="control-group">
        <?php echo $form->labelEx($model,'phone',array('class'=>'control-label')); ?>
        <div class="controls">
            <?php echo $form->textField($model,'phone',array('maxlength'=>250, 'style' => 'width: 359px;'));?>
            <?php echo $form->error($model,'phone'); ?>
        </div>
    </div>
    <div class="control-group">
        <?php echo $form->labelEx($model,'email',array('class'=>'control-label')); ?>
        <div class="controls">
            <?php echo $form->textField($model,'email',array('maxlength'=>250, 'style' => 'width: 359px;'));?>
            <?php echo $form->error($model,'email'); ?>
        </div>
    </div>
    <div class="control-group">
        <?php
        echo $form->labelEx($model, 'Territory Zip codes',array('class'=>'control-label'));
        ?>
        <div class="controls">
            <?php echo $form->dropDownList($model, 'zipcode', 0,array('class' => 'load-zip-code', 'id'=>'e2_2')); ?>
            <?php echo $form->error($model,'zipcode'); ?>
        </div>
    </div>
    <div class="control-group">
        <?php
        echo $form->labelEx($model, 'Factory',array('class'=>'control-label'));
        ?>
        <div class="controls">
            <?php echo $form->dropDownList($model, 'factory', 0,array('class' => 'load-factory', 'id'=>'factory')); ?>
            <?php echo $form->error($model,'factory'); ?>
            <?php echo $form->error($model, 'supervisor_id'); ?>
        </div>
    </div>

    <div class="control-group">
        <?php
        $superVisor = Supervisor::model()->findAll('status = :status', array(':status'=>Supervisor::STATUS_ACTIVE));
        echo $form->labelEx($model, 'Rep Supervisor',array('class'=>'control-label'));
        ?>
        <div class="controls">
            <?php echo $form->dropDownList(new Supervisor(), 'id', CHtml::listData($superVisor, 'id', 'name'), array('class' => 'form-control')); ?>
            <?php echo $form->error($model, 'supervisor_id'); ?>
        </div>
    </div>
    <div class="control-group">
        <?php echo $form->labelEx($model,'status',array ('class' => 'control-label')); ?>
        <div class="controls">
            <?php echo $form->dropDownList($model,'status',$model->getStatusOptions()); ?>
            <?php echo $form->error($model,'status'); ?>
        </div>
    </div>

    <div class="control-group">
        <?php  echo $form->labelEx($model, 'Email User Details',array('class'=>'control-label'));?>
        <div class="controls">
            <?php echo CHtml::checkBox('SalesRepresentative[emailUserDetails]',true,array());            ?>
        </div>
    </div>


    <div class="controls buttons">
        <?php echo CHtml::submitButton($model->isNewRecord ? 'Create' : 'Save',array('class' => 'btn btn-primary')); ?>
    </div>

    <?php $this->endWidget(); ?>

</div>

<script>
    function loadFactory(zipCodeId)
    {
        $.ajax({
            url: '<?php echo Yii::app()->createUrl("Hospitality/admin/salesRepresentative/loadfactory")?>',
            method: 'post',
            dataType: 'json',
            cache:'true',
            delay: 250,
            data: {id: zipCodeId},
            traditional: true,
            success: function(dataResponse) {
                $(".load-factory").html('').select2({data: {id:null, text: null}});
                $(".load-factory").select2({
                    data: dataResponse.result,
                    placeholder: "Select a factory",
                    allowClear: true,
                    escapeMarkup: function (markup) { return markup; }, // let our custom formatter worksss
                    templateResult: formatFactory, // omitted for brevity, see the source of this page
                    templateSelection: formatFactorySelection // omitted for brevity, see the source of this page
                })
            },
            error: function(result) {
                alert("error");
            }
        })
    }

    var total_rows = '<?php echo $totalCount; ?>';

    $(".load-zip-code").select2({
        placeholder: "Select a factory",
        allowClear: true,
        ajax: {
            url: '<?php echo Yii::app()->createUrl("Hospitality/admin/salesRepresentative/loadzipcode")?>',
            dataType: 'json',
            method: 'post',
            delay: 250,
            data: function (params) {
                return {
                    key: params.term, // search term
                    page: params.page,
                };
            },
            processResults: function (data, params) {
                // parse the results into the format expected by Select2.
                // since we are using custom formatting functions we do not need to
                // alter the remote JSON data
                params.page = params.page || 1;
                return {
                    results: data.result,
                    pagination: {
                        more: (params.page * 100) <  total_rows
                    },
                };
            },
            cache: true
        },
        escapeMarkup: function (markup) { return markup; }, // let our custom formatter work
        minimumInputLength:0,
        templateResult: formatTerritory, // omitted for brevity, see the source of this page
        templateSelection: formatTerritorySelection // omitted for brevity, see the source of this page

    });
    $(".load-zip-code").on("select2:select", function (e)
    {
        loadFactory(e.params.data.id)

    });
    $("#select2-e2_2-container").html("Select Territory Zip Codes");
    function formatTerritory (repo) {
        if (repo.loading) return repo.zipcode;

        var markup = '<div class="clearfix">' +
            '<div class="col-sm-6">' + repo.zipcode + ' - ' + repo.city  + ' - '+ repo.state+ '</div>' +
            '</div>';
        markup += '</div></div>';

        return markup;
    }

    function formatTerritorySelection (repo) {
        return repo.zipcode;
    }
    function formatFactory (repo) {
        if (repo.loading) return repo.name;
        console.log(repo);
        var markup = '<option value="'+repo.id+ '">' +repo.name+ '</option>';
        return markup;
    }
    function formatFactorySelection (repo) {
        return repo.name;
    }
</script>
